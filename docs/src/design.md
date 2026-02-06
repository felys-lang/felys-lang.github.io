# High-Level Design

Programming languages essentially translate logic expressed in natural language into operations executable by a Turing machine, so there are many ways to design them. Felys stands on the shoulders of giants — namely Rust and Python. However, being either too ambitious or too simplistic might lead to project failure by my definition, so this section reviews some programming language fundamentals and discusses the trade-offs I made when building Felys.

## Compiled or Interpreted

The key to determining whether a language is compiled or interpreted is to check if it eventually translates programs into machine code. Machine code refers to the executable that runs directly without requiring an external runtime. Traditional languages like C/C++ and modern ones like Rust and Go are well-known examples.

Most interpreted languages also use a compilation step for faster loading and better performance. For example, Python's compiler is lightweight: it mostly parses code and outputs bytecode. It cannot do many optimizations due to dynamic typing. In contrast, the Java compiler is heavier and performs many optimizations.

> Originally, Felys was purely interpreted by recursively traversing the syntax tree. However, this was slow because I didn't have a powerful runtime to perform garbage collection, and it required many hash maps to store variables. Therefore, I introduced the Felys compiler and a runtime to address these issues. You might wonder why I didn't target a compiled backend: the answer is simple — I didn't have time to learn LLVM.

## Explicit or Garbage Collected

Many programming languages use garbage collection to hide the complexity of memory management from programmers, although implementations can vary significantly. For example, Java relies on the JVM (Java Virtual Machine), which manages its own stack, heap, program counter, and other resources. In contrast, Go uses a built-in runtime and does not rely on an external platform.

C, C++, and Rust are some popular languages that do not require a garbage collection runtime. The former usually requires manual memory management (although C++ supports smart pointers), while the latter uses a borrow checker and lifetime analysis to ensure memory safety at compile time. Explicit memory management is usually faster and more suitable for embedded systems due to its determinism.

> Felys does neither because all objects are immutable, so reference counting provides acceptable performance. Implementing a garbage collector in Rust is not simple and might require `unsafe` operations. I would rather implement such systems in C. Therefore, the most controllable approach is to make everything immutable, and this decision aligns with the next subsection.

## Functional or Imperative

Functional programming is what many CS students learn in their first year, and it has been trending in recent years. Pure functional programming has no side effects, and everything is immutable. One benefit is that it makes logic clear and easier to debug, although it is not very beginner friendly. Languages of this type, such as Haskell and Racket, are commonly used in academia.

In contrast, imperative languages focus on flow control, introducing constructs like if-else clauses and loops. They are generally more readable for everyday use. To take advantage of functional programming, imperative languages often include partial support for it. This allows programmers to use functional thinking to write logic and easily assemble components to build maintainable and complex products.

> Felys is, in fact, very functional. As mentioned, this implies easier memory management and allows the compiler to perform many in-depth analyses. This enables Felys to have syntax similar to Rust and support many compile-time checks, so it doesn't need to introduce a `void` or `null` type, which could be troublesome.

## Static or Dynamic Typing

Static typing means that a variable is bound to a specific data type at compile time and cannot be reassigned to a value of a different type. In contrast, dynamic typing allows variables to hold values of different types during runtime without explicit declarations. If we look deep enough into how computers work, we find that data types are an abstraction used by higher-level languages. At the hardware level, data is just a sequence of bits. The processor has no concept of types and only executes instructions. It’s the assembler and compiler that ensure data is interpreted correctly, e.g., IEEE 754. When we arrange data in contiguous memory and define how it should be interpreted, we create data structures. This close relationship between memory layout and type interpretation makes static typing natural in many compiled languages, where knowing the type is essential for generating correct machine code.

As interpreted languages became more popular, dynamic typing emerged as a more flexible alternative because the runtime can handle types dynamically. One major benefit of dynamic typing is its flexibility and speed during development, especially in scripting and prototyping. But for large projects, developers often reintroduce type systems, e.g., Python or TypeScript.

> Felys is dynamically typed simply because it's very complicated to introduce a full type system. That would require new syntax, significant changes to the IR structs, and implementing a type inference mechanism, which is non-trivial. However, I would like to have one because I enjoyed Rust's approach — it's very elegant.

## Strong or Weak Typing

The key difference between strong and weak typing is type coercion. Some languages, such as JavaScript and C/C++, automatically convert data types when performing operations. For example, in C, the expression `1 + '1'` is valid because the character `'1'` is implicitly converted to its corresponding ASCII value. It's important not to confuse weak typing with operator overloading, as they are entirely different mechanisms. Weak typing can sometimes lead to unexpected behavior, so modern programmers tend to avoid relying on implicit conversions and instead write them out explicitly to improve code clarity and maintainability.

> Weak typing could be nice to have, but it's very prone to error. I don't like it, so I don't do it.
