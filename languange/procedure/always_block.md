# Always Block
Since digital circuits are composed of logic gates connected with wires, any circuit can be expressed as some combination of modules and assign statements. However, sometimes this is not the most convenient way to describe the circuit. Procedures (of which always blocks are one example) provide an alternative syntax for describing circuits.

For synthesizing hardware, two types of always blocks are relevant:

- Combinational: always @(*)
- Clocked: always @(posedge clk)

## Combinational: always @(*)
Combinational always blocks are equivalent to assign statements, thus there is always a way to express a combinational circuit both ways. The choice between which to use is mainly an issue of which syntax is more convenient. **The syntax for code inside a procedural block is different from code that is outside. Procedural blocks have a richer set of statements (e.g., if-then, case), cannot contain continuous assignments**, but also introduces many new non-intuitive ways of making errors. (*Procedural continuous assignments do exist, but are somewhat different from continuous assignments, and are not synthesizable.*)

For example, the assign and combinational always block describe the same circuit. Both create the same blob of combinational logic. Both will recompute the output whenever any of the inputs (right side) changes value.

For example, the assign and combinational always block describe the same circuit. Both create the same blob of combinational logic. Both will recompute the output whenever any of the inputs (right side) changes value.
```verilog
assign out1 = a & b | c ^ d;
always @(*) out2 = a & b | c ^ d;
```
For combinational always blocks, always use a sensitivity list of `(*)`. Explicitly listing out the signals is error-prone (if you miss one), and is ignored for hardware synthesis. If you explicitly specify the sensitivity list and miss a signal, the synthesized hardware will still behave as though `(*)` was specified, but the simulation will not and not match the hardware's behaviour. (In SystemVerilog, use always_comb.)

A note on `wire` vs. `reg`: The left-hand-side of an assign statement must be a net type (e.g., `wire`), while the left-hand-side of a procedural assignment (in an always block) must be a variable type (e.g., `reg`). These types (wire vs. reg) have nothing to do with what hardware is synthesized, and is just syntax left over from Verilog's use as a hardware simulation language.

the below `out_assign` and `out_alwaysblock` are identical
```verilog
module top_module(
    input a,
    input b,
    output wire out_assign,
    output reg out_alwaysblock
);
	assign out_assign = a && b;
    always @(*) out_alwaysblock = a && b;
endmodule
```

## Clocked: always @(posedge clk)
Clocked always blocks create a blob of combinational logic just like combinational always blocks, but also creates a set of flip-flops (or "registers") at the output of the blob of combinational logic. Instead of the outputs of the blob of logic being visible immediately, the outputs are visible only immediately after the next (posedge clk).
### Blocking vs Non-Blocking Assignment
There are three types of assignments in Verilog:

- Continuous assignments (`assign x = y;`). Can only be used when not inside a procedure (**always block**).
- Procedural blocking assignment: (`x = y;`). Can only be used inside a procedure.
- Procedural non-blocking assignment: (`x <= y;`). Can only be used inside a procedure.

In a **combinational** always block, use **blocking** assignments. In a **clocked** always block, use **non-blocking** assignments. A full understanding of why is not particularly useful for hardware design and requires a good understanding of how Verilog simulators keep track of events. Not following this rule results in extremely hard to find errors that are both non-deterministic and differ between simulation and synthesized hardware.

here `out_assign`, `out_always_comb` and `out_always_ff` will produce the same result except that `out_always_ff` will only get changed on positive edge of clock
```verilog
module top_module(
    input clk,
    input a,
    input b,
    output wire out_assign,
    output reg out_always_comb,
    output reg out_always_ff   );

    assign out_assign = a ^ b;
    always @(*) out_always_comb = a ^ b;
    always @(posedge clk) out_always_ff <= a ^ b;

endmodule
```
