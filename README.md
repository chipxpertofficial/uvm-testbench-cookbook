# UVM Testbench Cookbook

Practical, copy-adaptable **recipes for building UVM testbenches** — the patterns you reach for
again and again, explained with the *why*, not just boilerplate. Written for someone who knows
Verilog and is learning verification.

Canonical UVM needs a UVM-capable simulator (a commercial sim, or a supported open setup). The
recipes here are about **structure and methodology** — they transfer across simulators.

## Recipes

| Recipe | What you'll build |
|--------|-------------------|
| [01 · Anatomy of a UVM testbench](recipes/01-anatomy.md) | the component hierarchy and what each piece is for |
| [02 · A driver + sequence that actually talks](recipes/02-driver-sequence.md) | sequence_item, sequencer, driver handshake |
| [03 · Monitor, scoreboard & analysis ports](recipes/03-monitor-scoreboard.md) | checking correctness automatically |

(More recipes — RAL/register model, functional coverage, virtual sequences — added over time.)

## The mental model

A UVM testbench is a **pipeline of reusable components** connected by TLM ports:

```
test → env → agent ─┬─ sequencer → driver → (DUT pins)
                    ├─ monitor  → analysis port → scoreboard
                    └─ (config via uvm_config_db)
```

The point is reuse: the same agent/monitor/scoreboard works across many tests; only the
**sequences** (the stimulus) change per test. Get that separation right and everything else
follows.

## Who this is for

- You've written directed testbenches and want to move to constrained-random + self-checking.
- You're prepping for a verification role and need the structure to click.

## Companion material

- Verified RTL to verify against → [vlsi-verilog-examples](https://github.com/chipxpertofficial/vlsi-verilog-examples)
- Verification interview questions → [vlsi-interview-prep](https://github.com/chipxpertofficial/vlsi-interview-prep)
- Mentored verification training → [ChipXpert](https://chipxpert.in/)

## License

MIT — see [LICENSE](LICENSE). Maintained by the ChipXpert team.
