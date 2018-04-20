
# SOME CONCERNS ABOUT WEBASSEMBLY SECURITY AND DETERMINISM


Below are some concerns about WebAssembly execution in mission-critical situations.



## 1. SPEC-LEVEL

### IEEE 754-2008 Floating-point arithmetic

Floating-point operations may require special care for the following reasons.

(1) IEEE 754-2008 operations need not be associative, so modern architectures may allow out-of-order execution of floating-point instructions, which can round to different results. Some hardware has a bios switch to prevent out-of-order execution.

(2) `NaN` can to have an arbitrary non-zero significand. 

(3) IEEE 754-2008 instructions may not be the most efficient, so some hardware offers optional short-cuts. Some hardware does not even support IEEE754-2008.

A possible solution is to choose conformant hardware which executes instructions in-order, and to canonicalize `NaN`s. If that is not possible, it may be necessary to implement floating-point operations using integers.


### Resource Exhaustion

The Wasm spec does not define a limit on code size, execution stack size, or any other runtime object. So system resources can be exhausted. Memory size is bound by the 32-bit address space. A solution is to have a verification step to limit resources known at compile-time, and to have run-time checks which limit recursion depth.





## 2. EMBEDDING-LEVEL

The Wasm spec does _not_ define an embedding environment, API, or ABI for Wasm. The environment may not adhere fully to the spec.


### Validation

> Wasm Spec Section 3.4: Modules are valid when all components they contain are valid.

So a module with an invalid function is invalid.

> Wasm Spec Section 4.5.4: if module is not valid, then Fail.

The above excerpt refers to instantiation. So validity must be checked before execution.

> Wasm Spec Section 7.2.2: An implementation can defer validation of individual functions until they are first invoked. ... invalid functions result in a trap. ... the function must be validated before execution of its body begins.

This excerpt contradicts the previous one.


### Resource exhaustion errors

Errors for resource exhaustion may depend on the embedding environment and on the state of the system.


### Breaking out of VM sandbox

JIT spraying is writing arbitrary instructions to memory, then somehow redirecting execution to those instructions. The common solution is for the operating system to mark pages as writable xor executable ("W^X"). This is relevant to any VM which generates and executes binary code. For example, Firefox 46 (2016) implements a W^X policy.


### Asynchronous execution

Many Javascript implementations of Wasm allow modules to be instantiated and executed asynchronously, which may not be desired.


### JITs

A modern JIT can include an interpreter, a basic compiler, and an optimizing compilers. For example, Firefox starts with a basic single-pass compiler from Wasm to native assembly which allows execution to begin, while, in the background, the Ion compiler performs an optimized compilation.


Some JITs use profilers/tracers which heuristically trigger compilation of "hot" code. Compilation may involve an intermediate representation (IR) with optimization passes. These heuristics and optimizations may introduce bugs, or invariants which make it easy to introduce bugs. It may be difficult to prove correctness of something so complicated.






## 3. COMPILER-LEVEL


### Compiler bugs.

The translations, optimization passes, or register allocation algorithms may not preserve execution semantics. There are many subtleties in all of these areas.


### Undefined behavior.

A language with undefined behavior may be compiled to unwanted behavior in Wasm. Wasm only guarantees type-safety.





## 4. HARDWARE-LEVEL


### Hardware Bugs and Backdoors

Modern hardware is complicated (micro-ops, instruction queues, dependency chains, pipelining, etc). It is rumored that modern CPUs may not be fully tested. Bugs have been found. For mission-critical tasks, it may be wise to add redundancy with a variety of architectures.

http://danluu.com/cpu-bugs/

There may also be backdoors.

http://danluu.com/cpu-backdoors/



### Cosmic Rays

Cosmic rays may cause errors in electronics. The problem is worse with smaller transistors. ECC may be useful for RAM.



### Bottlenecks, Denial of Service

If arbitrary code is to be executed, an attacker can exploit bottlenecks in hardware, causing slow execution. For example, an attacker can exhaust cache, causing all memory instructions to go to RAM.




## 5. CONCLUSIONS

For mission-critical computation, it may be wise to add redundancy and variety in both hardware and software. And to use simple toolchains which can be auditted.

