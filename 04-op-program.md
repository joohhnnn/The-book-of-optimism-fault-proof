# op-program

op-program is mainly divided into two modules:

1. The Client module that provides ELF files.
2. The Host module that starts and maintains the pre-image.

## Client Module

The native implementations based on op-node and op-geth are complex and some operations cannot be represented by MIPS, hence they cannot directly generate corresponding ELF files. This necessitates a simplified version of op-node and op-geth that can fully express L2 state execution, remove complex operations, and fully adapt to MIPS. The Client module is a Golang module that aligns with production code logic, used to generate ELF files compatible with MIPS. These ELF files are later parsed by Cannon to produce state.json including memory and register layouts for Cannon's use.

### ELF Files
ELF (Executable and Linkable Format) files are widely used file formats, primarily for Unix and Unix-like operating systems (such as Linux). This format supports executable files, relocatable files, and shared libraries, and allows for detailed metadata, such as the memory layout of the program, positions and sizes of various segments (like code and data segments), and other information for dynamic linking and program execution.

When a program is compiled and packaged into an ELF file, its memory layout (such as the positions of code and data segments) and some initialization settings required at startup are fixed. In Cannon, these data are transformed into state.json, which is used as input for Cannon.

### Core Logic

The [runDerivation()](https://github.com/ethereum-optimism/optimism/blob/develop/op-program/client/program.go#L63) function implements the execution process of L2 state changes within the op-stack.

```golang
// runDerivation executes the L2 state transition, given a minimal interface to retrieve data.
func runDerivation(logger log.Logger, cfg *rollup.Config, l2Cfg *params.ChainConfig, l1Head common.Hash, l2OutputRoot common.Hash, l2Claim common.Hash, l2ClaimBlockNum uint64, l1Oracle l1.Oracle, l2Oracle l2.Oracle) error {
	l1Source := l1.NewOracleL1Client(logger, l1Oracle, l1Head)
	engineBackend, err := l2.NewOracleBackedL2Chain(logger, l2Oracle, l2Cfg, l2OutputRoot)
	if err != nil {
		return fmt.Errorf("failed to create oracle-backed L2 chain: %w", err)
	}
	l2Source := l2.NewOracleEngine(cfg, logger, engineBackend)

	logger.Info("Starting derivation")
	d := cldr.NewDriver(logger, cfg, l1Source, l2Source, l2ClaimBlockNum)
	for {
		if err = d.Step(context.Background()); errors.Is(err, io.EOF) {
			break
		} else if err != nil {
			return err
		}
	}
	return d.ValidateClaim(l2ClaimBlockNum, eth.Bytes32(l2Claim))
}
```

Note that this module is only responsible for compiling the ELF files, while the logic to parse ELF files into state.json is carried out in Cannon.

## Host Module
When Cannon starts, it also initiates the Host module (`./bin/op-program`, referring here to the binary name of the host after compilation, not the entire op-program module). When a syscall is executed and a read operation is required, a hint is thrown, and the Host service captures this hint and loads the data for use.

Since the Host is initiated with Cannon’s startup command, it acts as a submodule to Cannon. The two communicate via a pair of `FileChannel` objects, such as `pClientRW, pOracleRW`. The `pOracleRW` channel is passed to the subprocess during command creation and accessed via methods like `pWriter := os.NewFile(6, "pOracleWriter")`.

```golang
func NewProcessPreimageOracle(name string, args []string) (*ProcessPreimageOracle, error) {
	if name == "" {
		return &ProcessPreimageOracle{}, nil
	}

	pClientRW, pOracleRW, err := preimage.CreateBidirectionalChannel()
	if err != nil {
		return nil, err
	}
	hClientRW, hOracleRW, err := preimage.CreateBidirectionalChannel()
	if err != nil {
		return nil, err
	}

	cmd := exec.Command(name, args...) // nosemgrep
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	cmd.ExtraFiles = []*os.File{
		hOracleRW.Reader(),
		hOracleRW.Writer(),
		pOracleRW.Reader(),
		pOracleRW.Writer(),
	}
```

In the subprocess code, these descriptors are used:

```golang
// CreatePreimageChannel returns a FileChannel for the preimage oracle in a detached context
func CreatePreimageChannel() oppio.FileChannel {
	r := os.NewFile(PClientRFd, "preimage-oracle-read")
	w := os.NewFile(PClientWFd, "preimage-oracle-write")
	return oppio.NewReadWritePair(r, w)
}
```
Once communication is established, when Cannon needs to retrieve pre-image data for a specific key, it writes the key into the channel. The Host then reads the key from the channel, retrieves the corresponding data based on the key, and writes the data back into the channel for Cannon's use.

The core function is the [GetPreimage()](https://github.com/ethereum-optimism/optimism/blob/develop/op-program/host/prefetcher/prefetcher.go#L80) function, which receives the key passed from Cannon and retrieves the content.

```golang
func (p *Prefetcher) GetPreimage(ctx context.Context, key common.Hash) ([]byte, error) {
	p.logger.Trace("Pre-image requested", "key", key)
	pre, err := p.kvStore.Get(key)
	// Use a loop to keep retrying the prefetch as long as the key is not found
	// This handles the case where the prefetch downloads a preimage, but it is then deleted unexpectedly
	// before we get to read it.
	for errors.Is(err, kvstore.ErrNotFound) && p.lastHint != "" {
		hint := p.lastHint
		if err := p.prefetch(ctx, hint); err != nil {
			return nil, fmt.Errorf("prefetch failed: %w", err)
		}
		pre, err = p.kvStore.Get(key)
		if err != nil {
			p.logger.Error("Fetched pre-images for last hint but did not find required key", "hint", hint, "key", key)
		}
	}
	return pre, err
}
```

## Conclusion
- The official introduction to [OP-program](https://docs.optimism.io/stack/protocol/fault-proofs/fp-components#fault-proof-program) perfectly summarizes the functionality of the Client module: This program verifies the L2 output obtained from L1 inputs through rollup state transitions. This verifiable output can then be used to resolve disputes on L1. FPP is a combination of op-node and op-geth, thus it simultaneously possesses the protocol's consensus and execution "parts" within a single process. This means that typical Engine API calls made via HTTP are replaced with direct method calls to op-geth code. FPP was designed to operate deterministically, so that two calls made with the same input data not only produce the same output but also the same execution trace. This allows it to be used as part of the dispute resolution process, running in the on-chain VM.
- For the Host module, it serves as a service to provide pre-image information to Cannon during runtime.
