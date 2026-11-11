# 03 · Monitor, scoreboard & analysis ports

A testbench that drives but doesn't *check* is just a waveform generator. The monitor +
scoreboard are what make it self-checking — the thing that lets you run thousands of random tests
unattended.

## The monitor
The monitor is **passive**: it watches the DUT interface, reconstructs each transaction, and
broadcasts it on an `uvm_analysis_port`. It never drives anything.

```systemverilog
class bus_monitor extends uvm_monitor;
  virtual bus_if vif;
  uvm_analysis_port #(bus_item) ap;
  `uvm_component_utils(bus_monitor)
  function new(string name, uvm_component parent);
    super.new(name,parent); ap = new("ap", this);
  endfunction

  task run_phase(uvm_phase phase);
    forever begin
      @(posedge vif.clk);
      if (vif.valid) begin
        bus_item tr = bus_item::type_id::create("tr");
        tr.addr = vif.addr; tr.data = vif.data; tr.we = vif.we;
        ap.write(tr);           // broadcast to any subscriber
      end
    end
  endtask
endclass
```

## The analysis port / TLM
`ap.write(tr)` sends the transaction to **every** connected subscriber — one monitor can feed a
scoreboard *and* a coverage collector at once. Connection (in the env's `connect_phase`):
```systemverilog
agent.monitor.ap.connect(scoreboard.analysis_export);
agent.monitor.ap.connect(coverage.analysis_export);
```

## The scoreboard
The scoreboard holds a **reference model** (predicts the expected result) and compares it against
what the monitor observed.

```systemverilog
class bus_scoreboard extends uvm_scoreboard;
  `uvm_analysis_imp_decl(_dut)
  uvm_analysis_imp_dut #(bus_item, bus_scoreboard) analysis_export;
  bit [31:0] mem [bit[31:0]];   // the reference model
  `uvm_component_utils(bus_scoreboard)
  function new(string name, uvm_component parent);
    super.new(name,parent); analysis_export = new("analysis_export", this);
  endfunction

  function void write_dut(bus_item tr);
    if (tr.we) mem[tr.addr] = tr.data;             // model a write
    else begin                                     // check a read
      if (mem.exists(tr.addr) && mem[tr.addr] !== tr.data)
        `uvm_error("SCBD", $sformatf("addr %0h: exp %0h got %0h",
                    tr.addr, mem[tr.addr], tr.data))
    end
  endfunction
endclass
```

## Why this scales
Once the scoreboard encodes "what correct means," you can throw constrained-random sequences at
the DUT for hours — every mismatch is caught automatically. Add a **functional coverage** subscriber
on the same analysis port and you also learn *what you actually exercised*, closing the loop
between "it passed" and "it was tested."

## The three checks an interviewer will probe
1. **Correctness** — scoreboard compares against a reference.
2. **Completeness** — functional coverage says how much of the spec you hit.
3. **Did it even run** — end-of-test checks that stimulus was actually applied (a classic silent
   failure: 100% pass because nothing ran).
