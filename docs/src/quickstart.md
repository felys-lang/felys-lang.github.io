# Quickstart

The [playground](https://exec.felys.dev) also has some sample programs.

## Literal

Felys has built-in support for float, integer, string, tuple, list, and matrix. Their underlying implementation uses `f64`, `isize`, `String`, `Vec`, `Vec`, and `Vec` in Rust standard library.

```
int = 42;
float = 3.14;
string = "Elysia";
tuple = ("Elysia", 11.11);
list = [1, 2, 3, 4, 5];
matrix = [
    0.0, 0.0, 0.0;
    0.0, 0.0, 0.0;
];
learnable = <2, 3>;
```

## Comment

Just like many programming languages, Felys uses double slash for comments:

```
// This is a comment
```

## Assignment

Beyond standard assignment, you also unpack a tuple and the assign it. Felys also had syntactic sugar like `+=`, `-=`, etc.

```
(name, birthday) = ("Elysia", 11.0);
birthday += 0.11;
```

Felys also provides built-in identifier storing values in global scope. You can use `rust` keyword to distinguish them from user defined identifiers.

```
author = rust __author__;
```

## Operator

Just like any other languages, Felys has arithmetic, comparison, and logical operator. Be aware that Felys is strong typed, which means that things like `1` and `1.0`, are not same and `1 + 1.0` does not evaluate.

```
sum = 1 + 1;
dot = <1, 2> @ <2, 1>;
conjunction = true and true or false;
equality = 1 == 1;
```

## Flow control

Unlike condition in other languages, `else` in Felys allows any type of expression to follow by.

```
one = if true {
    1
} else 0;

one = if true {
    1
} else loop {
    break 1;
}
```

There are three types of loops using keywords `loop`, `while`, and `for`, along with `break` and `continue`. The `break` keyword appeared in `loop` can carry a return value. `for` loop will go through an iterable, i.e., list.

```
one = loop {
    break 1;
};

while true {
    if one {
        break;
    }
}

for x in [1, 2, 3] {
    if x == 2 {
        break;
    }
}
```

The last statement of a block is also the return value of the block. All statements before it must not have a return value. You can to use `;` to make them `void`, and the best practice is to always add the `;` except for expression ends with `}` and returns `void`.

```
one = { 1 };
```

## Statement

If semicolon shows up after an expression, this expression will have `void` return value, i.e. no return value. Most expressions have a return value except for assignment, `for` loop, `while` loop, `break`, `continue`, and `return`.

```
void = { 1; };
one = 1;
```

## Neural Network

All neural network related operation must be done on matrix containing float values only. Declaration of a matrix is similar to MATLAB:

```
matrix = [
    0.0, 0.0, 0.0;
    0.0, 0.0, 0.0;
];
```

If you want a learnable parameter that is initialized randomly, you can declare it like this:

```
matrix = <2, 3>;
```

The runtime can get the correct value when evaluating this, instead of initializing it again. For unmodified programs, you can optionally load previous parameters. If you are using the playground, this can be achieved by clicking the lock icon.

Once you want to back propagate, you can call:

```
step loss by 0.01;
```

The `loss` must be a matrix, and the `0.01` is the learning rate. Felys implemented a SGD optimizer with momentum as backend.

## Main

All statements in a Felys program must not have return values. The program has a default return value `void`, but you can return anything using the `return` keyword. This would also early terminates the program, and is also the only interface to output something.

```
return "Elysia";
```
