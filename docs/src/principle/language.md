# Programming Language

Before we get into any details, it is crucial to understand differences between languages.

## Compilation vs Interpretation

The key to determining whether a language is compiled or interpreted is to check if it is eventually translated into machine code. Machine code refers to the executable that runs directly without requiring an external runtime. Traditional languages like C/C++ and modern ones like Rust and Go are well-known examples.

Interpreted languages always require a runtime to execute the code, even if the source code goes through a compilation process. Python, Java, and JavaScript fall into this category. In terms of performance, compiled languages are usually much faster than interpreted ones, and some can reach speeds close to assembly language. However, Java can be an exception, as it is statically typed and benefits from advanced runtime optimizations, making its performance highly competitive.

Interpreted languages are usually more flexible and can easily integrate libraries written in compiled languages. For example, most machine learning libraries in CPython are actually written in C and C++, and CPython is just the interface.

## Explicit Memory Management vs Garbage Collection

Many programming languages use garbage collection to hide the complexity of memory management from programmers, although the implementations can vary significantly. For example, Java relies on the JVM (Java Virtual Machine), which manages its own stack, heap, program counter, and other resources. In contrast, Go uses a built-in runtime and does not rely on an external platform.

C, C++, and Rust are some popular languages that do not require a garbage collection runtime. The former usually requires manual memory management (although C++ supports smart pointers), while the latter uses a borrow checker and lifetime analysis to ensure memory safety at compile time. Explicit memory management is usually faster and more suitable for embedded systems due to its determinism.

## Functional vs Imperative

Functional programming is what many CS students learn in their first year, and it has been trending in recent years. Pure functional programming has no side effects, and everything is immutable. One benefit is that it makes logic clear and easier to debug, although it is not very beginner friendly. Languages of this type, such as Haskell and Racket, are commonly used in academia.

In contrast, imperative languages focus on flow control, introducing constructs like if-else clauses and loops. They are generally more readable for everyday use. To take advantage of functional programming, imperative languages often include partial support for it. This allows programmers to use functional thinking to write logic and easily assemble components to build maintainable and complex products.

## Static Typing vs Dynamic Typing

Static typing means that a variable is bound to a specific data type at compile time and cannot be reassigned to a value of a different type. In contrast, dynamic typing allows variables to hold values of different types during runtime without explicit declarations. If we look deep enough into how computers work, we find that data types are an abstraction used by higher-level languages. At the hardware level, data is just a sequence of bits. The processor has no concept of types and only executes instructions. It’s the assembler and compiler that ensure data is interpreted correctly, e.g., IEEE 754. When we arrange data in contiguous memory and define how it should be interpreted, we create data structures. This close relationship between memory layout and type interpretation makes static typing natural in many compiled languages, where knowing the type is essential for generating correct machine code.

As interpreted languages became more popular, dynamic typing emerged as a more flexible alternative as the runtime can handle types dynamically. One major benefit of dynamic typing is its flexibility and speed during development, especially in scripting and prototyping. But for large projects, developers often reintroduce type systems, e.g., Python or TypeScript.

## Strong Typing vs Weak Typing

The key difference between strong and weak typing is type coercion. Some languages, such as JavaScript and C/C++, automatically convert data types when performing operations. For example, in C, the expression `1 + '1'` is valid because the character `'1'` is implicitly converted to its corresponding ASCII value. It's important not to confuse weak typing with operator overloading, as they are entirely different mechanisms. Weak typing can sometimes lead to unexpected behavior, so modern programmers tend to avoid relying on implicit conversions and instead write them out explicitly to improve code clarity and maintainability.
