# 02 · A driver + sequence that actually talks

The driver↔sequencer handshake is where beginners stall. Here's the contract, in plain terms, plus
a skeleton you can adapt.

## The contract
1. A **sequence** creates a `sequence_item`, calls `start_item(req)`, randomises it, then
   `finish_item(req)`.
2. `finish_item` blocks until the **driver** has pulled the item via `seq_item_port.get_next_item(req)`.
3. The driver wiggles the DUT pins to carry out that transaction.
4. The driver calls `seq_item_port.item_done()` — which unblocks the sequence to send the next item.

That back-pressure (the sequence waits for the driver) is the point: stimulus is generated only as
fast as the driver can apply it.

## The sequence_item (the transaction)
```systemverilog
class bus_item extends uvm_sequence_item;
  rand bit [31:0] addr;
  rand bit [31:0] data;
  rand bit        we;     // write-enable
  `uvm_object_utils_begin(bus_item)
    `uvm_field_int(addr, UVM_ALL_ON)
    `uvm_field_int(data, UVM_ALL_ON)
    `uvm_field_int(we,   UVM_ALL_ON)
  `uvm_object_utils_end
  function new(string name="bus_item"); super.new(name); endfunction
endclass
```

## The driver skeleton
```systemverilog
class bus_driver extends uvm_driver #(bus_item);
  virtual bus_if vif;
  `uvm_component_utils(bus_driver)
  function new(string name, uvm_component parent); super.new(name,parent); endfunction

  function void build_phase(uvm_phase phase);
    if (!uvm_config_db#(virtual bus_if)::get(this,"","vif",vif))
      `uvm_fatal("NOVIF","virtual interface not set")
  endfunction

  task run_phase(uvm_phase phase);
    forever begin
      seq_item_port.get_next_item(req);   // pull from sequencer
      drive(req);                         // wiggle pins
      seq_item_port.item_done();          // release the sequence
    end
  endtask

  task drive(bus_item tr);
    @(posedge vif.clk);
    vif.addr <= tr.addr; vif.data <= tr.data; vif.we <= tr.we; vif.valid <= 1;
    @(posedge vif.clk);
    vif.valid <= 0;
  endtask
endclass
```

## The sequence skeleton
```systemverilog
class write_seq extends uvm_sequence #(bus_item);
  `uvm_object_utils(write_seq)
  function new(string name="write_seq"); super.new(name); endfunction
  task body();
    repeat (10) begin
      req = bus_item::type_id::create("req");
      start_item(req);
      assert(req.randomize() with { we == 1; });  // constrain to writes
      finish_item(req);
    end
  endtask
endclass
```

## Common snags
- **Null virtual interface** → `config_db` `set`/`get` name mismatch, or the `set` happened after
  the driver's `build_phase`.
- **Sequence runs but driver never fires** → the sequence wasn't started on the agent's sequencer
  (`seq.start(env.agent.sequencer)`), or the agent is passive.
- **Everything deadlocks** → you forgot `item_done()`.

Next: [monitor, scoreboard & analysis ports](03-monitor-scoreboard.md).
