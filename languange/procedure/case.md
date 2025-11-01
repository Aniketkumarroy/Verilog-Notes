# Always case
Case statements in Verilog are nearly equivalent to a sequence of if-elseif-else that compares one expression to a list of others. Its syntax and functionality differs from the switch statement in C.
```verilog
always @(*) begin     // This is a combinational circuit
    case (in)
      1'b1: begin
               out = 1'b1;  // begin-end if >1 statement
            end
      1'b0: out = 1'b0;
      default: out = 1'bx;
    endcase
end
```
- The case statement begins with case and each "case item" ends with a colon. There is no "switch".
- Each case item can execute exactly one statement. This makes the "break" used in C unnecessary. But this means that if you need more than one statement, you must use `begin ... end`.
- Duplicate (and partially overlapping) case items are permitted. The first one that matches is used. C does not allow duplicate case items.

case statements are useful when there is a large number of cases
```verilog
module top_module (
    input [2:0] sel,
    input [3:0] data0,
    input [3:0] data1,
    input [3:0] data2,
    input [3:0] data3,
    input [3:0] data4,
    input [3:0] data5,
    output reg [3:0] out   );

    always@(*) begin  // This is a combinational circuit
        case (sel)
            3'b000: out = data0;
            3'b001: out = data1;
            3'b010: out = data2;
            3'b011: out = data3;
            3'b100: out = data4;
            3'b101: out = data5;
            default: out = 4'b0000;
        endcase
    end

endmodule
```
## Priority Encoder using case
A priority encoder is a combinational circuit that, when given an input bit vector, outputs the position of the first 1 bit in the vector. For example, a 8-bit priority encoder given the input 8'b10010000 would output 3'd4, because bit[4] is first bit that is high.
```verilog
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );

    always @(*) begin
        case(1'b1)
            in[0]: pos = 2'b00;
            in[1]: pos = 2'b01;
            in[2]: pos = 2'b10;
            in[3]: pos = 2'b11;
            default: pos = 2'b00; // default case is strictly necessary because we haven't covered all possible input combinations
        endcase
    end

endmodule
```
we can also acheive the same functionality use `casex` statement and `x` state.
```verilog
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );

    always @(*) begin
        casex(in)
            4'bxxx1: pos = 2'b00; // in[3:1] can be anything we don't care
            4'bxx10: pos = 2'b01; // in[3:2] can be anything we don't care
            4'bx100: pos = 2'b10; // in[3:3] can be anything we don't care
            4'b1000: pos = 2'b11;
            default: pos = 2'b00;
        endcase
    end

endmodule
```
A case statement behaves as though each item is checked sequentially (in reality, a big combinational logic function). Notice how there are certain inputs (e.g., 4'b1111) that will match more than one case item. The first match is chosen (so 4'b1111 matches the first item, out = 0, but not any of the later ones).

a 8-bit priority encoder
```verilog
module top_module (
    input [7:0] in,
    output reg [2:0] pos );

    always @(*) begin

        casez (in)
            8'bzzzzzzz1: pos = 3'd0;
            8'bzzzzzz10: pos = 3'd1;
            8'bzzzzz100: pos = 3'd2;
            8'bzzzz1000: pos = 3'd3;
            8'bzzz10000: pos = 3'd4;
            8'bzz100000: pos = 3'd5;
            8'bz1000000: pos = 3'd6;
            8'b10000000: pos = 3'd7;
            default: pos = 3'd0;
        endcase
    end

endmodule
```
Suppose we are building a circuit to process scancodes from a PS/2 keyboard for a game. Given the last two bytes of scancodes received, you need to indicate whether one of the arrow keys on the keyboard have been pressed. This involves a fairly simple mapping, which can be implemented as a case statement (or if-elseif) with four cases.
```less
Scancode [15:0]	Arrow key
16'he06b	left arrow
16'he072	down arrow
16'he074	right arrow
16'he075	up arrow
Anything else	none
```
our circuit has one 16-bit input, and four outputs.

To avoid creating latches, all outputs must be assigned a value in all possible conditions. Simply having a default case is not enough. we must assign a value to all four outputs in all four cases and the default case. This can involve a lot of unnecessary typing. One easy way around this is to assign a "default value" to the outputs before the case statement:
```verilog
always @(*) begin
    up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;
    case (scancode)
        ... // Set to 1 as necessary.
    endcase
end
```
This style of code ensures the outputs are assigned a value (of 0) in all possible cases unless the case statement overrides the assignment. This also means that a default: case item becomes unnecessary.
```verilog
module top_module (
    input [15:0] scancode,
    output reg left,
    output reg down,
    output reg right,
    output reg up  );

    always @(*) begin
        left = 1'b0;
        right = 1'b0;
        down = 1'b0;
        up = 1'b0;
        case (scancode)
            16'he06b: left = 1'b1;
            16'he072: down = 1'b1;
            16'he074: right = 1'b1;
            16'he075: up = 1'b1;
        endcase
    end

endmodule
```