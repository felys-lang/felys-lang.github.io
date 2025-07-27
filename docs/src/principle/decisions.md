# Technical Decisions

It is necessary to clarify what is Felys built for, so that decisions make sense.

## What is Felys

Every modern programming language has its target. For example, Rust aims for memory safety without garbage collection, TypeScript adds typing to JavaScript, and Go targets simplicity and concurrency. It is not practical for a single person to develop and maintain an entire programming language, so at the very beginning, Felys does not aim at anything practical. Other than the reasons I stated in [stories](../stories.md) behind the screen, Felys is:

**A DEMONSTRABLE PROJECT TO ANYONE REVIEWING MY RESUME**

The keyword here is demonstrable which means that Felys should have a online playground where everyone can run and edit sample code. At the same time, I want Felys core package to be free from external dependencies to make it more challenging and also for gimmick purpose.

## Design and Reasoning

Corresponding to last section, Felys is designed to be interpreted, garbage collected, imperative, static typing, and strong typing.

### Interpreted and Garbage Collected

Interpreted languages typically offer greater control over resources, which allows me to make online playgrounds safer and more robust. Additionally, compilation would require Felys to either adapt to many different platforms or use LLVM as a backend. The first option is unstable and time-consuming, while LLVM represents an external dependency. Thus, interpretation and garbage collection are the only viable choices.

### Imperative

Many programmers, myself included, are not accustomed to functional programming. It's very common for people to forget it after taking an introductory course in their first year of study. To ensure that users of the online playground can understand and modify the sample code, Felys should be imperative and use syntax that people are familiar with.

### Static and Strong Typing

Enforcing static and strong typing will make programs less prone to silly mistakes and faster. The compiler can perform extensive checks to ensure type safety, allowing Felys to also support a powerful `match` statement. However, this feature is not yet supported, as it will be more natural to implement when building the custom virtual machine in the future.

## Choice of the Language

Rust is the chosen language for its type system, but there are many other alternatives. For example, C/C++ are popular choices because they are fast and allow direct memory management, while Go is another option due to its modern design and ease of use. Other interpreted languages like Python and TypeScript are not considered, as they are overly abstracted from low-level operations.

Rust is a very interesting language. There are countless modern features that I enjoy, such as the `match` statement, the `trait` system, and its functional statements. All of these make code clean and allow it to accurately express the underlying logic. However, the lifetime system can sometimes make things too complicated, eventually compromising that cleanliness. In general, the advantages outweigh the disadvantages, so Felys will continue using Rust until a better alternative emerges.
