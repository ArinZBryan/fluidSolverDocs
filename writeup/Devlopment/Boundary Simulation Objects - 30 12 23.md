
## Path A) - Pass `SimulationObject[]` to solver during tick function

### Method 1: Pass entire `SimulationObject[]`
- Use `in SimulationObject[]` or `ref SimulationObject[]` to ensure minimal memory transfers
	- Transfer a single memory address (8 bytes) vs entire array (massive) every frame/tick
- Requires filtering to ensure only `CollidableCell` instances are processed
	- `System.LINQ` would be useful (Use SQL-like syntax to do filtering)
	- Using `LINQ` expressions every frame / tick could be too resource intensive
- Minimal code changes needed to make this work (in theory)
### Method 2: Pass changes to `SimulationObject[]`
- Much more memory efficient (only pass bytes on object instantiation and destruction)
- Must ensure arrays are always kept up-to date with master
	- If not, then there will be un-rendered physics objects in the debug view, or phantom physics objects rendered but not simulated
- Requires potentially complex function to check for changes. This would also need to be quite fast (not fun to implement)
### Method 3: Pass pre-filtered `CollidableCell[]`
- Requires using `System.LINQ` every tick, these methods are fast but not *that fast*
- Pass array using `ref` or `in` arguments, so only a reference is used. 
	- Prefer `in` over `ref`, as the compiler does not need to involve the GC here
- Some code changes / refactoring needed, but nothing too major (I think)
## Path B) - Relocate `SimulationObject[]` to solver, and pass changes via function interface
### Method 1: Filter during tick function
- Requires constant `LINQ` usage. This could be costly.
### Method 2: Filter during array change
- How do we check array changes conveniently and not overdo the calculations
	- If using a `System.Collections.Generic.List<T>`, then changes to the list length might suffice
	- If using a `T[]`, then we might be able to convert it to a `System.Collections.Generic.Span<Byte>` and hash that using some cheap algorithm. It doesn't need to be a particularly good algorithm.
	- Perhaps use a set? or some hash map derived data structure?
## Path C) - Hybrid Approach
- Store only physics interactable in separate list/array on solver.
- Ensure all `CollidableCell` instances are added to both lists / arrays
	- As class instances are reference types, these are just collections of references. This sidesteps the issue of synchronising data (the pointers go to the same memory)
	- Perhaps the logic can just be put in the constructor, but that could be a bit clunky. Maybe just have functions where necessary, and do static analysis to ensure the instances are always handled properly.
- Only minor refactor, as most existing code can remain
- Data duplication is largely trivial, as we are only shallow copies of classes / structs, not deep copies

# I went with path C in the end