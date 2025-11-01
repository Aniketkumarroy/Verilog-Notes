# If else in Procedural Block
An if statement usually creates a multiplexer, selecting one input based on condition.
```less
             ___
            |   \
  x --------|    \
            |     |----- out
  y --------|    /
            |___/
              |
  sel --------|
```
```verilog
always @(*) begin
    if (condition) begin
        out = x;
    end
    else begin
        out = y;
    end
end
```
This is equivalent to using a continuous assignment with a conditional operator
```verilog
assign out = (condition) ? x : y;
```
> `Note` the procedural if statement provides a new way to make mistakes. The circuit is combinational only if `out` is always assigned a value.

here is a multiplexer
```verilog
module top_module(
    input a,
    input b,
    input sel_b1,
    input sel_b2,
    output wire out_assign,
    output reg out_always   );

    assign out_assign = ({sel_b1, sel_b2} == 2'b11) ? b : a;
    always @(*)
        begin
            if ({sel_b1, sel_b2} == 2'b11) out_always = b;
            else out_always = a;
        end
endmodule
```
```less
               ___
              |   \
  a ----------|0   \
              |     |----- out
  b ----------|1   /
              |___/
                |
              __|__
  sel_b1 ----|     |
  sel_b2 ----| AND |
              -----
```
## Avoid making Latches
When designing circuits, we must think first in terms of circuits:

- we want this logic gate
- we want a combinational blob of logic that has these inputs and produces these outputs
- we want a combinational blob of logic followed by a set of flip-flops

for example, a syntactically correct code can generate a wrong circuit
```less
If (cpu_overheated) then shut_off_computer = 1;
If (~arrived) then keep_driving = ~gas_tank_empty;
```
Syntactically-correct code does not necessarily result in a reasonable circuit (combinational logic + flip-flops). The usual reason is: "What happens in the cases other than those you specified?". Verilog's answer is: **Keep the outputs unchanged**.

This behaviour of **"keep outputs unchanged"** means the current state needs to be remembered, and thus produces a latch. Combinational logic (e.g., logic gates) cannot remember any state. Watch out for `Warning (10240): ...` inferring latch(es)" messages. Unless the latch was intentional, it almost always indicates a bug. Combinational circuits must have a value assigned to all outputs under all conditions. This usually means you always need else clauses or a default value assigned to the outputs.

The following code contains incorrect behaviour that creates a latch.
```verilog
always @(*) begin
    if (cpu_overheated)
       shut_off_computer = 1;
end

always @(*) begin
    if (~arrived)
       keep_driving = ~gas_tank_empty;
end
```
![alt text](/assets/images/if_else_latch.png)
the below code fix the issue
```verilog
module top_module (
    input      cpu_overheated,
    output reg shut_off_computer,
    input      arrived,
    input      gas_tank_empty,
    output reg keep_driving  );

    always @(*) begin
        if (cpu_overheated)
           shut_off_computer = 1;
        else
           shut_off_computer = 0;
    end

    always @(*) begin
        if (~arrived)
           keep_driving = ~gas_tank_empty;
        else
            keep_driving = 1'b0;
    end

endmodule
```
## Conditional Ternary Operator
Verilog has a ternary conditional operator ( ? : ) much like C:
```less
(condition ? if_true : if_false)
```
```verilog
(0 ? 3 : 5)     // This is 5 because the condition is false.
(sel ? b : a)   // A 2-to-1 multiplexer between a and b selected by sel.
```
This can be used to choose one of two values based on condition (a mux!) on one line, without using an if-then inside a combinational always block.

```verilog
always @(posedge clk)         // A T-flip-flop.
  q <= toggle ? ~q : q;

...

always @(*)                   // State transition logic for a one-input FSM
  case (state)
    A: next = w ? B : A;
    B: next = w ? A : B;
  endcase

...

assign out = ena ? q : 1'bz;  // A tri-state buffer

((sel[1:0] == 2'h0) ? a :     // A 3-to-1 mux
 (sel[1:0] == 2'h1) ? b :
                      c )
```
a circuit for finding minimum
```verilog
module top_module (
    input [7:0] a, b, c, d,
    output [7:0] min);

    wire[7:0] t1, t2;
    assign t1 = (a < b) ? a : b;
    assign t2 = (c < d) ? c : d;
    assign min = (t1 < t2) ? t1 : t2;

endmodule
```