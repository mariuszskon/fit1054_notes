# What this is

An inconsistent-in-quality collection of notes mainly aimed at FIT1054 Computer science (advanced), but since that is generally a superset of FIT1008 and FIT2085, it works for those too.

These notes are probably more useful to know what to revise rather than actually revising, as the units have a strong applied focus.

# MIPS

* You get a reference sheet so don't bother memorising specific registers, instructions, calling conventions
* General architecture
    * 32 general purpose registers of 32 bits
        * on reference sheet
    * Special registers
        * Instruction Register (IR)
        * Program Counter (PC)
        * Memory address/read/write (MAR/MRR/MWR)
    * Arithmetic Logic Unit
    * Memory
        * Byte addressible (4 byte words means you often multiply by 4)
        * Segments assign implicit meaning to words
        * OS segment
        * Text segment (executable code)
        * Data segment
            * Static data (known at compile time)
            * Heap segment (growing down, runtime allocatable)
            * Stack segment (growing up, used for function locals)
        * Then another OS segment
* Address dereferencing
    * In this unit, all of our arrays have the first element as the length, then the actual elemnts are 1 indexed
        * so allocate an extra word (4 bytes)
    * NOTE: always increment by a multiple of 4 (a word)
    * sw $t0, 4($sp) # mem[$sp + 4] = $t0
    * if you allocate array dynamically, you store the pointer, so use lw not la to retrieve it correctly
    * Classic array acess pattern
```
lw  $t0, the_list  # get pointer to start of list (actually the length "element")
lw  $t1, i         # load i
sll $t1, 2         # multiply i by 4 to get byte offset
add $t0, $t0, $t1  # $t0 is now &the_list[i-4]
lw  $t2, 4($t0)    # $t2 = the_list[i]
sw  $t3, 8($t0)    # the_list[i+1] = $t3
```

* Functions
    * In this unit, always use stack for arguments and locals (unless optimising)
    * Follow conventions in the reference sheet
    * Decrement $sp to allocate stack space - "push"
    * Increment $sp to deallocate - "pop"
    * Recursion - don't worry about it, stack deals with it
    * Classic memory diagram of a stack frame

| Register which points here | meaning for humans  | example value | how to access                     |
|----------------------------|---------------------|--------------:|-----------------------------------|
| $sp ->                     | (local) result      |           125 | -8($fp)                           |
|                            | (local) other_local |            42 | -4($fp)                           |
| $fp ->                     | (saved $fp)         |    0x7FFF03A0 | after deallocating locals, 0($sp) |
|                            | (saved $ra)         |    0x0040005C | after deallocating locals, 4($sp) |
|                            | (arg) base          |             5 | 8($fp)                            |
|                            | (arg) exp           |             3 | 12($fp)                           |
* Gotchas
    * .ascii "Hello" does not null terminate (when string ends is not specified), so please use .asciiz
    * You cannot use ALU operations (arithmetic, logic) on memory, you must load them into registers first, then store the results
    * When doing comparisons, you usually want the exact opposite e.g. say you want if (x > 2) {...} you want to jump after the block on condition !(2 < x) (slt then bne)
    * do not forget unconditional jumps at end of if for an if-else, or at end of loop
    * Function calling conventions in the reference sheet that are defined as "actually necessary" by this unit (aka. not "just conventions", do not hack around them):
        1. Caller:
            * save / restore temporary registers (on / from stack)
            * pass arguments on stack
            * call the callee function with `jal`
        2. Callee:
            * save / restore $ra
            * allocate / deallocate local variables
            * return with `jr $ra`
    * Function calling conventions in the reference sheet that are defined as "just conventions" by this unit (hacking is possible, but discouraged):
        1. Caller:
            * clear arguments off stack
            * use return value in $v0
        2. Callee:
            * sets $v0 to return value
            * save / restore $fp
* Compiler optimisations
    * Constant folding - evaluate constant expressions at compile time e.g. 24 * 365 => 8760
    * Bitwise/bitshifting operations
        * Multiplication by powers of two - use sll e.g. instead of x*8 do x << 3
        * Division by powers of two - use srl (logical) or sra (arithmetic)
        * AND 1 tells you if a number is odd
    * Reordering expressions to use fewer registers
    * Reusing registers instead of reading/writing from/to memory all the time
    * Loops
        * Changing loop conditions (if safe) (e.g. for (i = 0; i < 20; i++) and i not modified inside =\> for (i = 0; i != 20; i++))
        * Deferring loop conditions until after first iteration (e.g. for (i = 0; i < 20; ...))
    * Breaking functional calling conventions to reduce instructions, including passing arguments by registers, not using $fp, not using $ra (if safe)
    * Eliminating jumps so fetching suits pipeline (just get next instruction), branch prediction more effective
        * Duplicate code (copying blocks of code, inlining functions, loop unrolling)
        * Rearrange conditionals so most ocmmon branch first
        * Other operations
    * Inline functions
        * Calling functions has instruction (time) and stack (space) cost
        * You can just copy the function code in many cases
    * Tail recursion
        * If your algorithm has been implemented in a tail recursive friendly way you can reuse the same stack frame for each call because its contents will not be used when returning
* Evaluating compiler optimisations
    * Often a trade off of time vs speed
    * Trade off of efficiency and readability
    * Loss of reusability, increased chance of errors
    * Try to only optimise code which runs often, do not worry about code which runs very rarely
* Simple comparison

Python
```python
if x <= y:
    # Do something
else:
	# Do something else
```
MIPS
```

	# Given that x,y are both defined in the text segment
	lw $t0, x
	lw $t1, y
	
# Check comparison
	# (x <= y) == !(y < x) 
	slt $t2, $t1, $t0
	bne $t2, $0, else
if:
	# Do Something

else: 
	# Do Something
```

# Algorithms

* Sorting
    * n is number of element in the list
    * You might need to multiply complexity by comparison cost `m` e.g. if strings or other non-atomic data types

| Algorithm        | Overview                                                                                                     | Best                               | Worst        | Stable                  | Incremental                                                     |
|------------------|--------------------------------------------------------------------------------------------------------------|------------------------------------|--------------|-------------------------|-----------------------------------------------------------------|
| Bubble sort      | Repeatedly swap adjacent out-of-order elements                                                               | O(n^2), or O(n) if detecting swaps | O(n^2)       | Yes (strict inequality) | Yes (if element added to front, which sucks because it is O(n)) |
| Selection sort   | Repeatedly swap minimum element to front                                                                     | O(n^2)                             | O(n^2)       | No                      | No                                                              |
| Insertion sort   | Build a sorted sublist at the front                                                                          | O(n)                               | O(n^2)       | Yes (strict inequality) | Yes (if element is added to back, which is usually O(1))        |
| Merge sort       | Recursively merge sorted sublists                                                                            | O(n\*log(n))                       | O(n\*log(n)) | Yes                     | No                                                              |
| Quick sort       | Recursively pick pivot and put it in the right place (everything lower to left, everything greater to right) | O(n\*log(n))                       | O(n^2)       | No                      | No                                                              |
| Pqueue/heap sort | Form a heap out of elements and repeatedly get_max                                                           | O(n\*log(n))                       | O(n\*log(n)) | No                      | No                                                              |

* Recursion
    * Arity
        * Unary - one recursive call
        * Binary - two recursive calls (e.g. Fibonaci, merge sort)
        * n-ary - n recursive calls
    * Direct - function calls itself within itself
    * Indirect/mutual - function calls a different function which ends up calling the original function
    * Tail recursion - recursive call is last, no addition or other transformation
        * This can be optimised by compilers so you can reuse the same stack frame, basically making your recursion iterative
    * Converting iterative algorithm into recursive one
        * Possible, and generally straightforward
    * Converting recursive algorithm into iterative one
        * Possible, but more difficult (e.g. you might need to manage your own stack)
* Dynamic programming
    * Basically an optimisation over recursion so you don't recompute the same results over and over


| Algorithm                            | Overview                               | Best               | Worst              |
|--------------------------------------|----------------------------------------|--------------------|--------------------|
| Fibonacci                            | Build on previous two cells in array   | O(n)               | O(n)               |
| Longest sequence of positive numbers | Get longest sequence ending at index i | O(n)               | O(n)               |
| Maximum subsequence sum              | Get maximum sum ending at index i      | O(n)               | O(n)               |
| Knapsack                             | M[first_items, capacity]               | O(items\*capacity) | O(items\*capacity) |

# Data structures (abstract data types)

* Abstract data types
    * Possible values
    * Meaning of those values
    * Operations
    * Do not care about implementation
* Data types
    * Abstract data types but concretely implemented
    * Can either be primitive (built-in and usually quite simple e.g. `int`) or user-defined (e.g. our `LinkedQueue`)
* Data structures
    * A particular way in which data is physically organised memory

The following table is compiled based on how these ADTs were concretely implemented.
Unless specified otherwise, `n` means number of elements and `m` is cost of comparison (I assume ==, \< and \> all cost the same, the unit sometimes does not, notably when dealing with binary trees).
By keeping count of elements as you add/remove them, length check can be O(1).

| Abstract data type          | Brief description                                                                 | Operations (W= worst Big-O; B= best Big-O)                                                                                                                                                              |
|-----------------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| (Unsorted) List             | Basically a python list                                                           | Get item (W=B=O(1))<br>Set item (W=B=O(1))<br>Search (W=O(m\*n); B=O(m))<br>Add last/append (W=B=O(1))<br>Delete (by value) (W=B=O(m\*n))                                                               |
| Sorted List                 | List but in ascending order                                                       | Get item (W=B=O(1))<br>Set item (W=B=O(1))<br>Search (W=O(m\*log(n)); B=O(m))<br>Add (W=O(m\*n); B=O(m))<br>Delete (by value) (W=O(m\*n)); B=O(m\*log(n)))                                              |
| (Array) Stack               | LIFO with arrays                                                                  | Pop / get top (W=B=O(1))<br>Push / add to top (W=B=O(1))                                                                                                                                                |
| (Array, non-circular) Queue | FIFO with arrays, serving does not free space                                     | Serve / get front (W=B=O(1))<br>Append / add to back (W=B=O(1))                                                                                                                                         |
| (Array, circular) Queue     | FIFO with arrays, serving does free space by moving pointers                      | Serve / get front (W=B=O(1))<br>Append / add to back (W=B=O(1))                                                                                                                                         |
| Linked Stack                | LIFO with pointer to top, each node to node underneath                            | Pop / get top (W=B=O(1))<br>Push / add to top (W=B=O(1))                                                                                                                                                |
| Linked Queue                | FIFO with pointer to front and pointer to rear                                    | Serve / get front (W=B=O(1))<br>Append / add to back (W=B=O(1))                                                                                                                                         |
| Linked List                 | Basically a python list, pointer to front of list                                 | Get item (W=B=O(n))<br>Set item (W=B=O(n))<br>Search (W=O(m\*n); B=O(m))<br>Add last/append (W=B=O(n))<br>Add front/prepend (W=B=O(1))<br>Insert (W=O(n); B=O(1))<br>Delete (by index) (W=O(n); B=O(1)) |
| (Hash Table) Dictionary     | Mapping of keys to values, unordered                                              | Search (W=O(m\*n); B=O(m))<br>Insert (W=O(m\*n); B=O(m))<br>Delete (W=O(m\*n); B=O(m))                                                                                                                  |
| Binary Search tree          | Binary tree where left < node < right, recursively<br>depth (W=O(n); B=O(log(n))) | Search (W=O(m\*depth); B=O(1))<br>Insert (W=O(m\*depth); B=O(1))<br>Delete (W=O(m\*depth); B=O(1))<br>                                                                                                  |
| (Max Heap) Priority queue   | Complete binary tree where parent is larger than children, recursively            | Add (W=O(m\*log(n)); B=O(m))<br>get_max (W=O(m\*log(n)); B=O(m))                                                                                                                                        |

* Data structures which were touched on very very briefly
    * Circular linked list - linked list but last element points to first
    * Double linked list - each node not only points to next one but also previous one

* Benefits of using abstract data types (ADTs)
    * Provides info regarding possible values and their meaning
    * Simplicity - only the operations are exposed, not the inner workings
    * Reusability - they are quite generic and can be reused in many different situations
    * Flexibility - different implementations can be used interchangeably

* Arrays
    * Backbone of many data types, particularly list-like ones
    * Fixed length!
    * You may need to keep track of size of your data type within the actual allocated space
* Trees
    * Graphs (as in graph theory)
    * Has nodes connected by edges
    * No circuits i.e. one unique path between any two notes
    * We usually choose a node to be the root
    * Leaves are nodes without any children
    * Binary trees
        * Have at most two children
        * Perfect binary trees have 2^(k+1)-1 nodes
        * Height of balanced tree is O(log(n))
        * Height of unbalanced tree (e.g. "stick") is O(n)
        * Balance is calculated as `|height(left\_subtree) - height(right\_subtree)| <= 1`
        * Binary tree traversal
           * Preorder - root, then recurse left, recurse right
           * Inorder - recurse left, root, recurse right
           * Postorder - recurse left, recurse right, root
        * Each x-order leads to x-fix notation for mathematical expressons, where postfix is Reverse Polish Notation
* Objects
    * Everything in python is an object
    * You have attributes (variables of this instance) and methods (functions which can modify these attributes)
    * They can point to other objects to created linked structures
    * Very useful for implementing basically all data types we cover

* Linked data structures
    * Instead of a contiguous array, you use objects which point to each other
    * Advantages:
        * Fast insertion and deletion because no need for reshuffling
        * Easily resizable
        * Never full
        * Less memory allocated in advante (i.e. better if array would be relatively empty)
    * Disadvantages:
        * More memory used (constant factor) compared to relatively full array because need to store pointers
        * Generally no random access, making certain operations more time consuming
    * Optimisations
        * Self organising linked lists - most recently accessed elements are likely to be accessed again - this encourages the best case when searching
            * MTF (move to front)
            * Transposition (swap one place closer)
            * Both of these cost O(1) due to relinking
        * Unrolling linked lists
            * Construct a linked list which consists of arrays which actually hold each element
            * This saves space on memory pointers
            * May improve access times to due locality thanks to caching

* Heaps
    * We implement them as arrays starting at index 1 (so index 0 wasted)
    * To get left child, 2\*k
    * To get right child, 2\*k + 1
    * To get parent for a node for k > 1, k//2
    * Remember to choose the largest child when sinking
    * If we build heap bottom-up, and then repeatedly extract maximum and place it in the "hole" created by the extraaction (at the end of array), we get a sorted list (aka heap sort)
    * Bottom up heap construction involves going from right of array and working leftwards to heap-ify
        * Start at last non-leaf and apply the sink procedure
        * This takes O(m\*n) time
```
for i in range(max_size//2, 0, -1):
    self.sink(i)
```

* Hash tables
    * Usually allow for O(1) key lookup, as long as collisions do not occur
    * Collision resolution
        * If different key produces same hash, we still need to deal with the unique elemnt
        * Same the key alongside value to ensure it is the right key
        * Approaches:
            * Open addressing - each array position holds a single element, if collision occurs, choose another one
                * For each of these, i means the iteration of the "try again" loop, starting at 0
                * If some other key is already there, repeat the loop
                * If it is the same key, overwrite the value
                * Open addressing could lead to clustering
                    * Primary clustering - "position collisions" even when hashes do not collide
                    * Secondary clustering - keys with same hash values have same probe chains
                * You need to take care to "wrap around" if your incrementing operations exceed last allowable index
                * Perform iteration at most count times where count is number of inserted elements, to prevent going into infinite loop if table full (or ensure your table is never full)
                * Common implementations:
                    * Linear probing - hash+i
                        * 1/2 load
                    * Quadratic probing - hash+i^2, or add 2i+1 at the end of each iteration (odd numbers produce squares)
                        * 1/2 load
                    * Double hashing - hash + (i+1)\*another\_hash (another\_hash is the same key under a different hash function)
                        * 2/3 load
            * Separate chaining - each array contains a linked list (or some other data structure) of items, if collision occurs, add to this data structure
                * Conceptually simpler because items with same hash stored in the same array slot
                * Naturally resizes on collisions
                * Insertions and deletions are dealt with by the data structure
                * But extra space required for overhead such as links in the case of linked lists
                * Complexity of data structures then determines worst case
        * Deletion in open addressing
            * Lazy deletion - mark an element as "deleted" but so probe chains know not to stop and claim another element does not exist
            * Rehashing - walk along until next free slot and reinsert these elements into the table
        * Maintaining efficient size
            * Keep load factor (filled slots / table size) under recommended values above
            * When need to expand, double size
                * But it might be extra smart to ensure prime size, or at least odd, to minimise effects of modulo
            * When shrinking, halve size
            * On resizing, you need to rehash to ensure things are in the expected locations
    * Hash functions
        * Convert a key into an index for lookup in an array
        * Return value which is always in range of 0 to last allowable index
        * deterministic (same input always yields same output)
        * Good if it is fast
        * Good if it minimises collisions (multiple different keys producing same hash)
            * Collisions are usually unavoidable, particularly when mapping large sets onto small sets e.g. all strings to numbers 0-1000000
        * Perfect hash functions have no collisions, mapping every key into unique postion, but these rely on unique property of allowable keys and requires knowing the entire set in advance
        * Good functions approximate random functions rather than trying to be perfect
        * For a uniform hash, chance of collision is 1/tablesize
    * "the" good enough hash funtion for this unit
        * deals with permutations of the same letters
        * is straightforward to implement without being terrible in terms of collisions
        * you **should*** use a prime table sie and prime base to ensure values are spread out well
```
def hash(word):
    value = 0
    for i in range(len(word)):
        value = (value*BASE + ord(word[i])) % tablesize
    return value
```


# Misc

* Python variable representation
    * Your variable names do not hold the actual values
    * They hold pointer to actual values
    * This is why lists are mutable - your variable name still points to the same thing
        * Be careful
    * Garbage collection means python automatically frees up space with no pointers to it
    * Static scoping means that structure of code determines which variable names override each other
    * Block scoping means local variables are defined only within their "blocks"
    * Know how classes affect scoping
* Assertions
    * Check preconditions which must not be violated in between your own code
    * Do not use this to validate user input - this is an "expected" violation of requirements but not precondition
    * AssertionError raised if False
* Exceptions
    * Use in "exceptional" cases e.g. invalid user input
* Unit testing
    * Test valid and invalid cases of code
    * Ensure correct results produced
    * Ensure correct exceptions raised
* Iterable
    * Class which defines \_\_iter\_\_(self)
    * Returns a class (iterator) which defines \_\_next\_\_(self)
        * This maintains state
        * Return next element if it exists
        * Raises `StopIteration` when no more elements
        * This exception is automatically caught when you do `for x in my_iterator` and tells the for loop to stop
* Birthday paradox / birthday problem
    * The somewhat unintuitive problem that as number of people increases, probability that any two share the same birthday increases rapidly
    * Assuming there are 365 possible birthdays of equal probability
    * 23 people have a >50% chance
    * 50 people have >95% chance
    * 100 people have >99.99% chance
    * n people have `1 - (365 \* 364 \* 363 \* ... \* (365 - n - 1))/(365^n)` chance
    * This is relevant to the unit when dealing with hash collisions in hash tables
* Array optimisations
    * Shuffling
        * You can minimise writes by holding on to the element you want to insert and only save it at the end rather than performing repeated swaps
    * Using integers as lists of booleans
        * bit lists - 0 or 1 for false/true, using bitwise operators to read/write, which saves a lot of space
    * Flattening
        * If you have a multi-dmensional array, you can use arithmetic to access element in a particular spot in a one-dimensional array
            * e.g. 2d matrix into array means `\[row\]\[col\]` becomes `\[width*row+col\]` (width is number of columns)
    * Resizing
        * Double size whenever capacity exceeded
