# Vectors
Vectors must be declared:
```verilog
type [upper:lower] vector_name;
```

type specifies the datatype of the vector. This is usually wire or reg. If you are declaring a input or output port, the type can additionally include the port type (e.g., input or output) as well. Some examples:
```verilog
wire [7:0] w;         // 8-bit wire
reg  [4:1] x;         // 4-bit reg
output reg [0:0] y;   // 1-bit reg that is also an output port (this is still a vector)
input wire [3:-2] z;  // 6-bit wire input (negative ranges are allowed)
output [3:0] a;       // 4-bit output wire. Type is 'wire' unless specified otherwise.
wire [0:7] b;         // 8-bit wire where b[0] is the most-significant bit.
```
The endianness (or, informally, "direction") of a vector is whether the the least significant bit has a lower index (`little-endian, e.g., [3:0]`) or a higher index (`big-endian, e.g., [0:3]`). In Verilog, once a vector is declared with a particular endianness, it must always be used the same way. e.g., writing vec[0:3] when vec is declared wire [3:0] vec; is illegal. Being consistent with endianness is good practice, as weird bugs occur if vectors of different endianness are assigned or used together.
## Assigning Vector Elements
Accessing an entire vector is done using the vector name. For example:
```verilog
wire [3:0] a;
wire [7:0] w;
assign w = a;
```
takes the entire 4-bit vector `a` and assigns it to the entire 8-bit vector `w` (declarations are taken from above). If the lengths of the right and left sides don't match, it is zero-extended or truncated as appropriate.

The part-select operator can be used to access a portion of a vector:
```verilog
w[3:0]      // Only the lower 4 bits of w
x[1]        // The lowest bit of x
x[1:1]      // ...also the lowest bit of x
z[-1:-2]    // Two lowest bits of z
b[3:0]      // Illegal. Vector part-select must match the direction of the declaration.
b[0:3]      // The *upper* 4 bits of b.
assign w[3:0] = b[0:3];    // Assign upper 4 bits of b to lower 4 bits of w. w[3]=b[0], w[2]=b[1], etc.
```
## Implicit nets
Implicit nets are often a source of hard-to-detect bugs. In Verilog, net-type signals can be implicitly created by an assign statement or by attaching something undeclared to a module port. Implicit nets are always one-bit wires and causes bugs if you had intended to use a vector. Disabling creation of implicit nets can be done using the `default_nettype none directive.
```verilog
wire [2:0] a, c;   // Two vectors
assign a = 3'b101;  // a = 101
assign b = a;       // b =   1  implicitly-created wire
assign c = b;       // c = 001  <-- bug
my_module i1 (d,e); // d and e are implicitly one-bit wide if not declared.
                    // This could be a bug if the port was intended to be a vector.
```
Adding  **`default_nettype none** none would make the second line of code an error, which makes the bug more visible.
```verilog
`default_nettype none     // Disable implicit nets. Reduces some types of bugs.
module top_module();
    wire [2:0] a, c;   // Two vectors
    assign a = 3'b101;  // a = 101
    assign b = a;       //❌ Illegal: can't create implicit wire
endmodule
```
## Bitwise operators
there are bitwise operator which takes two inputs of same length and perform the operation bit by bit
```verilog
module top_module(
    input [2:0] a,
    input [2:0] b,
    output [2:0] out_or_bitwise,
    output out_or_logical,
    output [5:0] out_not
);

    assign out_or_bitwise = a | b;
    assign out_or_logical = a || b;
    assign out_not = {~a, ~b};
endmodule
```
other bitwise operator and their logical form is `&`(bitwise) and `&&`(logical). `^`(bitwise xor) don't have any logical counterpart.
`&`, `|` and `^` can also be operated on single vector variable, they just behave like a gate with inputs as all the bits of the vector and 1 bit output.
```verilog
module top_module(
    input [3:0] in,
    output out_and,
    output out_or,
    output out_xor
);
	assign out_and = &in;
    assign out_or = |in;
    assign out_xor = ^in;
endmodule
```
## Concatenation
`{}` is the concatenation operator
```verilog
module top_module (
    input [4:0] a, b, c, d, e, f,
    output [7:0] w, x, y, z );

    assign {w, x, y, z} = {a, b, c, d, e, f, 2'b11};

endmodule
```
will output
```less
 ____________________________
| a | b | c | d | e | f | 11 |
 ----------------------------
              |
 ____________________________
|   w  |  x   |  y   |   z   |
 ----------------------------
```
reversing the bit ordering of a vector
```verilog
module top_module (
	input [7:0] in,
	output [7:0] out
);

	assign {out[0], out[1], out[2], out[3], out[4], out[5], out[6], out[7]} = in;

endmodule
```
## Replication Operator
`{num{vector}}` stacks `vector` `num` times to create a new element
```verilog
{5{1'b1}}           // 5'b11111 (or 5'd31 or 5'h1f)
{2{a,b,c}}          // The same as {a,b,c,a,b,c}
{3'd5, {2{3'd6}}}   // 9'b101_110_110. It's a concatenation of 101 with
                    // the second vector, which is two copies of 3'b110.
```
One common place to see a replication operator is when sign-extending a smaller number to a larger one, while preserving its signed value. This is done by replicating the sign bit (the most significant bit) of the smaller number to the left. For example, sign-extending `4'b0101 (5)` to 8 bits results in `8'b00000101 (5)`, while sign-extending `4'b1101 (-3)` to 8 bits results in `8'b11111101 (-3)`.
```verilog
module sign_extender (
    input [7:0] in,
    output [31:0] out );

    assign out = {{24{in[7]}}, in};

endmodule
```