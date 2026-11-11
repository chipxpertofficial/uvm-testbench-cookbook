# 01 · Anatomy of a UVM testbench

Before writing a line, know what each component is *for*. UVM's hierarchy looks heavy at first;
every piece earns its place once you see the job it does.

## The components, top to bottom

| Component | Job |
|-----------|-----|
| `uvm_test` | the top: picks the configuration and starts a sequence. One per test scenario. |
| `uvm_env` | container that wires agents + scoreboard together. Reusable across tests. |
| `uvm_agent` | bundles one interface's sequencer + driver + monitor. Can be **active** (drives) or **passive** (only observes). |
| `uvm_sequencer` | arbitrates and hands sequence_items to the driver. |
| `uvm_driver` | converts transactions into pin wiggles on the DUT interface. |
| `uvm_monitor` | observes the pins, reconstructs transactions, broadcasts them on an analysis port. |
| `uvm_scoreboard` | receives observed transactions and checks correctness. |
| `uvm_sequence` / `uvm_sequence_item` | the **stimulus** — what you vary per test. |

## Why the split matters
The **structure** (env/agent/monitor/scoreboard) is built once and reused. The **stimulus**
(sequences) changes per test. That separation is the whole value of UVM: a new test is usually a
new sequence, not a new testbench.

## Active vs passive agent
- **Active** — has a driver + sequencer; it *drives* the interface. Use on the DUT's inputs.
- **Passive** — monitor only; it *watches*. Use on outputs, or when another block drives the bus.
Switching an agent passive (via its config) lets the same code observe an interface it used to
drive — useful at integration.

## The phases (the part that confuses everyone first)
UVM runs components through ordered **phases**: `build` (construct children top-down),
`connect` (wire TLM ports bottom-up), `run` (the time-consuming simulation), then report phases.
Build top-down, connect bottom-up — remember that and 90% of "my handle is null" bugs disappear.

## `uvm_config_db` — how settings reach deep components
Instead of passing handles through every constructor, you `set` a value (a virtual interface, a
config object, an `is_active` flag) at the top and `get` it where needed. It's UVM's dependency
injection. Most "virtual interface is null" errors are a `set`/`get` string mismatch.

Next: [a driver + sequence that actually talks](02-driver-sequence.md).
