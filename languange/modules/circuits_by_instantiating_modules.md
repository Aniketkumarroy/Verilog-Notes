# Hierarchical Modelling
This is hardware modelling style where the behaviour of a complex circuit is implemented by instantiating smaller modules.
## 3 bit shift register using D-Flip Flop
```verilog
module top_module ( input clk, input d, output q );
    wire q0, q1;
    d_ff mod0( .clk(clk), .d(d), .q(q0) );
    d_ff mod1( .clk(clk), .d(q0), .q(q1) );
    d_ff mod2( .clk(clk), .d(q1), .q(q) );
endmodule
```
![alt text](/assets/images/len_3_shift_reg.png)
```less
        |--------------------------X--------------------------|
        |                          |                          |
        |     ______________       |     ______________       |     ______________
        |    |              |      |    |              |      |    |              |
clk ----X----|>clk          |      |----|>clk          |      |----|>clk          |
             |              |           |              |           |              |
             |              |           |              |           |              |
d   ---------|d            q|-----------|d            q|-----------|d            q|---- output
             |              |    q0     |              |     q1    |              |
              --------------             --------------             --------------
```
## 8-bit wide shift register of length 3 using D-Flip Flop
Instead of module ports being only single pins, we now have modules with vectors as ports, to which we can attach wire vectors instead of plain wires. Like everywhere else in Verilog, the vector length of the port does not have to match the wire connecting to it, but this will cause zero-padding or trucation of the vector.
In addition to that we will also have a 4x2 mux to select what output to get
```verilog
module top_module (
    input clk,
    input [7:0] d,
    input [1:0] sel,
    output [7:0] q
);
    wire [7:0] q0, q1, q2;
    d_ff8 mod0(.clk(clk), .d(d), .q(q0));
    d_ff8 mod1(.clk(clk), .d(q0), .q(q1));
    d_ff8 mod2(.clk(clk), .d(q1), .q(q2));

    mux4x2 mod3(.in0(d), .in1(q0), .in2(q1), .in3(q2), .out(q), .sel(sel));
endmodule
```
```less
          |--------------------------X--------------------------|
          |                          |                          |
          |     ______________       |     ______________       |       ______________
          |    |              |      |    |              |      |      |              |
clk ------X----|>clk          |      |----|>clk          |      |------|>clk          |
               |              |           |              |             |              |
               |              |   q0      |              |    q1       |              |   q2
d   ----/----X-|d            q|----/----X-|d            q|-----/----X--|d            q|---/----|
        8    | |              |    8    | |              |     8    |  |              |   8    |
             |  --------------          |  --------------           |   --------------         |
             |                          |                           |                          | 8   _______
             |                          |                           |              8           |-/--|in3    |
             |                          |              8            |--------------/----------------|in2 out|--/----output
             |             8            |--------------/--------------------------------------------|in1    |  8
             |-------------/------------------------------------------------------------------------|in0    |
                                                                                                     -------
        2                                                                                               |
sel ----/-----------------------------------------------------------------------------------------------|
```
![alt text](/assets/images/len_3_8bit_shift_reg.png)
# 32-bit adder using 16 bit adder modules
```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire carry, c;
    add16 lower(.a(a[15:0]), .b(b[15:0]), .cin(1'b0), .sum(sum[15:0]), .cout(carry));
    add16 upper(.a(a[31:16]), .b(b[31:16]), .cin(carry), .sum(sum[31:16]), .cout(c));
endmodule
```
```less

                               ______________                  ______________
                              |              |     carry      |              |
                      0 ------|cin       cout|----------------|cin       cout|---
     32           a[15:0]     |              |                |              |
a ----/-----X--------/--------|a          sum|--/-----|  |----|a          sum|--/----|
b ----/-----|-X------/--------|b             |  16    |  | |--|b             |  16   |
     32     | |   b[15:0]     |              |        |  | |  |              |       |sum[31:16]
            | |      a[31:16]  --------------         |  | |   --------------        |
            |-|----------/----------------------------|--| |                         |
              |----------/----------------------------|----|                         |    32
                     b[31:16]                         |------------------------------X----/---- sum
                                                                     sum[15:0]
```
![alt text](/assets/images/32bit_adder_using_16bit_adders.png)
# Carry Select Adder
```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire carry, c0, c1;
    wire [15:0] sum0, sum1;
    add16 lower(.a(a[15:0]), .b(b[15:0]), .cin(1'b0), .sum(sum[15:0]), .cout(carry));
    add16 upper0(.a(a[31:16]), .b(b[31:16]), .cin(1'b0), .sum(sum0), .cout(c0));
    add16 upper1(.a(a[31:16]), .b(b[31:16]), .cin(1'b1), .sum(sum1), .cout(c1));

    assign sum[31:16] = (carry == 1'b0) ? sum0 : sum1;
endmodule
```
```less

                               ______________                  ______________                          ______________
                              |              | carry          |              |                        |              |
                      0 ------|cin       cout|-----|    0 ----|cin       cout|---               1 ----|cin       cout|----
     32           a[15:0]     |              |     |          |              |                        |              |  16
a ----/-----X--------/--------|a          sum|--/--|-|  |-----|a          sum|--/----|           |----|a          sum|--/----|
b ----/-----|-X------/--------|b             |  16 | |  | |---|b             |  16   |           | |--|b             |       |
     32     | |   b[15:0]     |              |     | |  | |   |              |       |sum0       | |  |              |       |sum1
            | |      a[31:16]  --------------      | |  | |    --------------        |           | |   --------------        |
            |-|----------/-------------------------|-|--X-|--------------------------|-----------| |                         |   _______
              |----------/-------------------------|-|----X--------------------------|-------------|                         |--|in1    | sum[31:16]         32
                     b[31:16]                      | |                               |                                          |    out|---/-----------X----/----sum
                                                   | |                               |------------------------------------------|in0    |               |
                                                   | |                sum[15:0]                                                  -------                |
                                                   | |------------------/-----------------------------------------------------------|-------------------|
                                                   |--------------------------------------------------------------------------------|
```
![alt text](/assets/images/32bit_carry_select_adder_using_16bit_adders.png)
## Adder Subtractor
```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    input sub,
    output [31:0] sum
);

    wire carry, co;
    wire [32: 0] c;
    assign c = b ^ {32{sub}};
    add16 lower(.a(a[15:0]), .b(c[15:0]), .cin(sub), .sum(sum[15:0]), .cout(carry));
    add16 upper(.a(a[31:16]), .b(c[31:16]), .cin(carry), .sum(sum[31:16]), .cout(co));

endmodule
```
```less

                                                  ______________                  ______________
                                                 |              |     carry      |              |
                                         0 ------|cin       cout|----------------|cin       cout|---
                        32           a[15:0]     |              |                |              |
a -----------------------/-----X--------/--------|a          sum|--/-----|  |----|a          sum|--/----|
b ----|   _______  c |---/-----|-X------/--------|b             |  16    |  | |--|b             |  16   |
      |--|in  out|---|  32     | |   c[15:0]     |              |        |  | |  |              |       |sum[31:16]
          -------              | |      a[31:16]  --------------         |  | |   --------------        |
sub ---------|                 |-|----------/----------------------------|--| |                         |
                                 |----------/----------------------------|----|                         |    32
                                        c[31:16]                         |------------------------------X----/---- sum
                                                                                        sum[15:0]
```
![alt text](/assets/images/32bit_adder_subtractor_using_16bit_adders.png)