# Module
a `module`, which is a circuit that interacts with its outside through input and output ports. Larger, more complex circuits are built by composing bigger modules out of smaller modules and other pieces (such as assign statements and always blocks) connected together. This forms a hierarchy, as modules can contain instances of other modules.
```verilog
module mod_a (
    input in_1,
    input in_2,
    output out_1,
    input in_k,
    ...
    output out_2,
    ...
    output out_p,
    );

    // Module body
endmodule
```
```less
     ______________
----|in_1          |
----|in_2     out_1|----
----|..       out_2|----
    |..          ..|..
    |..       out_p|----
----|in_k          |
     --------------
```
The hierarchy of modules is created by instantiating one module inside another, as long as all of the modules used belong to the same project (so the compiler knows where to find the module). The code for one module is not written inside another module's body (Code for different modules are not nested).
## Connecting Signals to Module Ports
There are two commonly-used methods to connect a wire to a port: by position or by name.

- By position
The syntax to connect wires to ports by position should be familiar, as it uses a C-like syntax. When instantiating a module, ports are connected left to right according to the module's declaration. For example:
```verilog
mod_a instance1 ( wa, wb, wc, .. );
```

This instantiates a module of type mod_a and gives it an instance name of **instance1**, then connects signal `wa` (outside the new module) to the first port (`in_1`) of the new module, `wb` to the second port (`in_2`), and `wc` to the third port (`out_1`). One drawback of this syntax is that if the module's port list changes, all instantiations of the module will also need to be found and changed to match the new module.

- By name
Connecting signals to a module's ports by name allows wires to remain correctly connected even if the port list changes. This syntax is more verbose, however.
```verilog
mod_a instance2 ( .out_1(wc), .in_1(wa), .in_2(wb) );
```
The above line instantiates a module of type mod_a named **instance2**, then connects signal `wa` (outside the module) to the port named `in_1`, `wb` to the port named `in_2`, and `wc` to the port named `out_1`. Notice how the ordering of ports is irrelevant here because the connection will be made to the correct name, regardless of its position in the sub-module's port list. Also notice the period immediately preceding the port name in this syntax.
> `Note`
> ```verilog
> mod_a instance(...)
> ```
> will result in a error because `instance` is a reserved keyword in verilog. though I don't know what is the use of it or why is it reserve.
