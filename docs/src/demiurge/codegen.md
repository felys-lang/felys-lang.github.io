# Lazy Compilation

The final compilation is nothing more than running the compilation pipeline and bringing the pieces together. However, it is actually more complicated than I thought. To minimize the output binary, we only compile reachable functions and groups. All functions should remain as syntax trees without any transformation until other functions call them, so that we do not waste time computing unnecessary information.

This algorithm performs multiple tasks at once but requires only one pass:

- Update and record reachable groups and compile their methods
- Compile reachable functions starting from main
- Record used constants
- Identifiers mapped to corresponding indices
- Variables mapped to registers

## Worklist Design

The solution I'm about to present needs two worklists for groups and functions separately and a constants pool. It can avoid direct recursion and is more maintainable. We have seen the pooling implementation several times, so I will skip it and focus on the worklist implementation.

```rust,noplayground
struct Worker<T> {
    indices: HashMap<usize, Index>,
    source: HashMap<usize, T>,
    worklist: Vec<(Index, T)>,
}
```

Unlike normal worklists, the `Worker` struct carries all the source with it, i.e., all candidates are already there. However, they only get removed and added to the worklist when we need them for the first time. Simultaneously, they receive a self-incrementing `Index` used by the IR to map identifiers to indices. This also handles self-recursion.

```rust,noplayground
impl<T> Worker<T> {
    fn get(&mut self, id: usize) -> Index {
        if let Some(index) = self.indices.get(&id) {
            return *index;
        }
        let index = Index::try_from(self.indices.len()).unwrap();
        self.indices.insert(id, index);
        let todo = self.source.remove(&id).unwrap();
        self.worklist.push((index, todo));
        index
    }

    fn pop(&mut self) -> Option<(Index, T)> {
        self.worklist.pop()
    }
}
```

## Full Compilation Flow

A struct `Context` wraps around the worklists and constant pool, just to keep things clean:

```rust,noplayground
struct Context {
    data: Data,
    groups: Worker<Group>,
    functions: Worker<(Vec<usize>, Block)>,
}
```

We first compile the main function and then continuously pop groups and functions. For groups, we also need to update the `methods` because they originally point to identifiers; we need indices now. All reachable groups and functions are collected in other hash maps. We cannot use a `Vec` here because the pop order usually does not match the actual order. However, it is guaranteed that all `Index` values (starting from `0`) will be collected in those hash maps. Below is a shortened implementation.

```rust,noplayground
let mut groups = HashMap::new();
let mut callables = HashMap::new();

let main = compile(args, block, &mut context, ..)?;

while !context.done() {
    while let Some((index, group)) = context.groups.pop() {
        for id in group.methods.values_mut() {
            *id = context.functions.get(*id)
        }
        groups.insert(index, group, &mut context, ..);
    }

    while let Some((index, (args, block))) = context.functions.pop() {
        let callable = compile(args, block)?;
        callables.insert(index, callable, &mut context, ..);
    }
}
```

Once it's all done, we iterate from `0` and remove entries until nothing remains. We create a helper function `linearize` here:

```rust,noplayground
fn linearize<T>(map: HashMap<Index, T>) -> Vec<T> {
    let mut i = 0;
    let mut all = Vec::new();
    while let Some(value) = map.remove(&i) {
        all.push(value);
        i += 1;
    }
    all
}
```

It is suggested to check if any thing still left in the map to catch implementation mistakes. However, as you might have noticed, Felys entirely relies on correctness of algorithms, and never do defensive programming for elegancy.

## IR to Bytecode

It's too verbose to show the transformation because many of them are exactly one-to-one relations, except for details like mapping variables to registers, constants/groups/functions to indices. You can image how clean it would be when we have these worklists and pooling.
