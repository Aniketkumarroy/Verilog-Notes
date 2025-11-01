# Vectors in Verilog

a vector in verilog can be defined in one of the two ways:

- Packed array

```verilog
// <Datatype> [<MSB>:<LSB>] var;
wire [7:0] byte_vector;         // packed array of 8 bits (a vector)
wire [3:0][7:0] data_bus;       // 4 elements of 8 bits each (total 32 bits)
```

- Unpacked array

```verilog
// <Datatype> var[<MSB>:<LSB>]
reg bit_memory [0:15];   // unpacked array of 16 elements, each 1 bit
reg [7:0] memory [0:15];   // unpacked array of 16 elements, each 8 bits
```

| Aspect                | **Packed Array**                                             | **Unpacked Array**                                                                         |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| **Purpose**           | Represents **contiguous bits**, like a wide vector.          | Represents **a collection of separate elements**, each of which can be packed or unpacked. |
| **Bit layout**        | Stored **contiguously in a single vector** (bit-accurate).   | Each element stored **independently**, not bit-contiguous.                                 |
| **Declaration order** | Declared **before** the variable name.                       | Declared **after** the variable name.                                                      |
| **Indexing order**    | **Right-most index varies fastest** (MSB→LSB, like vectors). | **Left-most index varies fastest**, like standard C arrays.                                |
| **Bit-level ops**     | Supports bit-slicing, part-select, concatenation.            | Cannot do bit-slicing across elements; access elements one by one.                         |
| **Base Datatype**     | can be both **net**(wire, tri, etc) and **reg**.             | can be only **reg**                                                                        |
| **Synthesis**         | Represents hardware like a **bus or register**.              | Represents **memories, arrays of registers**, or FIFOs.                                    |
---

## Packed Array
we can do part select or bit select
```verilog
wire [31:0] bus;
assign bus[7:0]   = 8'hAA;     // part-select
assign bus[15]    = 1'b1;      // bit-select
```
Multi-dimensional packed arrays behave like flattened bit vectors in synthesis and assignment:
```verilog
reg [1:0][3:0] two_by_four; // 2 elements, each 4 bits = total 8 bits

initial begin
  two_by_four = 8'b1010_1100; // Assigns as if it's a single 8-bit vector
end
```
Packed arrays are synthesizable and used heavily to model buses, registers, or packed fields.
## Unpacked Arrays
```verilog
reg [7:0] memory [0:15];   // unpacked array of 16 elements, each 8 bits
reg [3:0] array2d [0:7][0:1];  // 8×2 unpacked elements, each 4-bit packed
```
This behaves like a memory or array of registers, not a single wide bus.
```verilog
memory[0] = 8'hA5;   // Write first element
memory[1] = 8'h5A;
```
we cannot slice through multiple elements but we can bit select a single element
```verilog
memory[0][3:0][2:1] // ❌ Illegal: can't slice across multiple unpacked elements
memory[0][3:0] // ✅ lower nibble of first element
```
## Indexing and Ordering
- Packed Arrays
Indexing is from MSB to LSB, like vectors.
```verilog
wire [3:0] vec;   // 4 bits
// vec[3] is MSB, vec[0] is LSB
```
- Unpacked Arrays
Indexing is like C arrays — each dimension is accessed separately.
```verilog
reg [7:0] arr [0:3];
arr[0] = 8'h11;
arr[1] = 8'h22;
```
## Synthesis
| Type           | Typical Hardware Interpretation                |
| -------------- | ---------------------------------------------- |
| Packed array   | Single **bus / register** with wide bit width. |
| Unpacked array | **Array of registers**, like a small RAM.      |

```verilog
wire [32:0] mem_flat;    // 1 element of 32 bits packed into 32-bit vector;
wire [3:0][7:0] mem;     // 4 elements of 8 bits packed into 32-bit vector;
```
Synthesizes to a single 128-bit register.
```less
mem_flat = [ 32-bit element ]
mem = [ 8-bit element3 | 8-bit element2 | 8-bit element1 | 8-bit element0 ]
```
```verilog
reg [7:0] reg_file [0:31];   // 32 x 8-bit memory
```
will get synthesized to a 32 x 8-bit register file
```less
reg_file
 ├─ [0] : 8 bits
 ├─ [1] : 8 bits
 ...
 └─ [31]: 8 bits
```
**Multi-dimensional** packed arrays flatten into a single vector for assignments.
**Multi-dimensional** unpacked arrays remain separate memory-like structures.
