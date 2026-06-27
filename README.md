# Pipelined RISC-V Processor

A 5-stage pipelined RISC-V processor built in a digital logic simulator. The
design executes RISC-V instructions across overlapping pipeline stages and
includes a hazard unit that resolves data and control hazards automatically —
no manually inserted `nop` instructions required.

## Architecture

The processor implements the classic 5-stage RISC pipeline. Each stage is
separated by pipeline registers that carry instruction state forward on every
clock edge.

```
  IF        ID        EX        MEM       WB
 ┌────┐   ┌────┐   ┌────┐   ┌────┐   ┌────┐
 │Inst│──▶│Dec │──▶│ ALU│──▶│ Mem│──▶│ Reg│
 │Fetch│  │+ RF│   │+Br │   │ R/W│   │Write│
 └────┘   └────┘   └────┘   └────┘   └────┘
    ▲        ▲        ▲
    │        │        │
    └────────┴────────┴──── Hazard Unit (stall / forward / flush)
```

| Stage | Responsibility |
|-------|----------------|
| **IF** — Instruction Fetch | Reads the instruction at the program counter and increments the PC. |
| **ID** — Decode | Decodes the instruction and reads operands from the register file. |
| **EX** — Execute | Performs ALU operations and resolves branch targets/conditions. |
| **MEM** — Memory | Reads from or writes to the load/store memory subsystem. |
| **WB** — Writeback | Writes results back into the register file. |

## Hazard Handling

The hazard unit is the core of the project. It detects and resolves the three
classes of pipeline hazard:

- **Data hazards — register forwarding.** When an instruction needs a result
  that a prior instruction has computed but not yet written back, the value is
  forwarded directly from the EX/MEM or MEM/WB stage instead of stalling.
- **Load-use hazards — stalling.** A load followed immediately by a dependent
  instruction can't be resolved by forwarding alone, since the value isn't
  available until after MEM. The hazard unit inserts a one-cycle stall (bubble).
- **Control hazards — flushing.** On a taken branch, instructions already
  fetched along the wrong path are flushed so they never commit.

## Components

- **Register file** — 32 general-purpose registers with dual read ports and a
  single write port.
- **ALU** — arithmetic and logic operations driven by the control unit.
- **Branch unit** — evaluates branch conditions and computes targets.
- **Load/store memory subsystem** — word-addressable RAM with read/write
  support.
- **Hazard unit** — forwarding, stall, and flush control across the pipeline.

## Testing

The design was validated against three progressive test suites, each targeting
a specific hazard class:

1. **Arithmetic / forwarding** — back-to-back dependent ALU instructions.
2. **Load-use stalling** — a load immediately followed by a dependent use.
3. **Control-hazard flushing** — branches that redirect the instruction stream.

All three suites pass with the hazard unit handling dependencies automatically,
demonstrating correct execution at full pipeline throughput.

## Tooling

- **ISA:** RISC-V
- **Built in:** a digital logic / circuit simulator (`.dig` schematic format)
- **Top-level module:** `project07.dig`

## Background

Built as the capstone of a computer architecture course that progressed from
RISC-V assembly and machine code, through processor emulation, into digital
design — register files, ALUs, branch units, and memory — culminating in this
fully pipelined implementation.
