# SOME CONCERNS ABOUT WEBASSEMBLY SECURITY AND DETERMINISM

For mission-critical code execution, it is good to be aware of possible security and determinism risks. Below is a list of some concerns about WebAssembly execution.



## SPEC-LEVEL

### IEEE 754-2008 Floating-point arithmetic


(1) IEEE 754-2008 operations need not be associative, so modern architectures can allow out-of-order execution of floating-point instructions, which can round to different results. 

(2) `NaN` can to have an arbitrary non-zero significand. 

(3) Some hardware offers less expensive floating-point instruction by default. Some hardware does not even support IEEE754-2008.


### Resource Exhaustion

The Wasm spec does not define a limit on code size, execution stack size, or any other object the runtime must maintain. So system resources can be exhausted. Memory size is bound by the 32-bit address space.





## EMBEDDING-LEVEL

### Embedding environment

The Wasm spec does not define an embedding environment or an ABI for Wasm. For example, the Javascript API allows modules to be executed by asynchronous methods, which may not be desired.


### Breaking out of VM sandbox

CPU executable memory may also be writable. "JIT spraying" is writing arbitrary instructions to memory, then somehow redirecting execution to those instructions. The common solution is for the operating system to mark pages as writable xor executable ("W^X"). For example, Firefox 46 (2016) implements a W^X policy.



### JITs

Modern JITs are based on interpreters, compilers to assembly, and optimizing compilers with intermediate representation (IR) with optimization passes. For example, Firefox starts with a single-pass translation from Wasm to native assembly which allows execution to begin, while, in the background, the Ion compiler is busy performing an optimized compilation.

Some JITs use profilers/tracers which heuristically trigger optimizations when a chunk of code becomes "hot".

Optimizations and heuristics may introduce bugs, and may be difficult to prove correct.






## COMPILER-LEVEL

### Compilers bugs.

The translations, optimization passes, or register allocation algorithms may not preserve execution semantics, there are many subtleties.


### Undefined behavior.

A language with undefined behavior may be compiled to unwanted behavior in Wasm.





## HARDWARE-LEVEL

### Hardware Bugs and Backdoors

Modern hardware may not be fully tested. Bugs have been found. For mission-critical things, it may be wise to add redundancy with a variety of architectures.

http://danluu.com/cpu-bugs/

http://danluu.com/cpu-backdoors/



### Cosmic Rays

Cosmic rays may cause errors in electronics. The problem is worse with smaller transistors. ECC may be useful, but only for RAM. NASA uses redundancy and large transisitors.



### Bottlenecks, Denial of Service

An attacker can exploit bottlenecks in hardware, causing slow execution. For example, an attacker can exhaust cache causing all memory instructions to go to RAM.

