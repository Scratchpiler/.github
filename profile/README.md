<div align="center">

<img src="https://raw.githubusercontent.com/Scratchpiler/scratchpiler/main/resources/Scratchpiler%202.PNG" alt="Scratchpiler" width="220" />

# S C R A T C H P I L E R

### we put a compiler toolchain inside a website for children and we are not sorry

<sub>(we are a little sorry)</sub>

</div>

---

## hello. sit down. this is an org.

Somewhere, a ten-year-old is dragging a yellow `move 10 steps` block into a script.
Somewhere else, a grown adult is staring at that same block, thinking **"there has to be a better way."**

There was not.
So we made one.
Then we made a *second* one that sits underneath the first one.
Then we got a fuzzer involved and things went to a place.

**Scratchpiler** is two projects in a trench coat pretending to be an organization:

|  |  |
|---|---|
| <img src="https://raw.githubusercontent.com/Scratchpiler/scratchpiler/main/resources/Scratchpiler.png" width="160" alt="Scratchpiler logo" /> | **[scratchpiler](https://github.com/Scratchpiler/scratchpiler)**: a Monaco editor (yes, *the* VS Code one) bolted into the Scratch editor by a Tampermonkey userscript. Write real text. Press `Ctrl+Enter`. Blocks appear in the sprite like digital weeds. |
| <img src="https://raw.githubusercontent.com/Scratchpiler/slvm/main/resources/slvm-logo.png" width="160" alt="SLVM: a cat riding the LLVM dragon" /> | **[slvm](https://github.com/Scratchpiler/slvm)**: the **S**cratch **L**evel **V**irtual **M**achine. An LLVM parody with an actual IR, an actual verifier and actual optimization passes. Every program you compile goes through it before becoming colored puzzle pieces. |

Yes, that is the Scratch cat riding the LLVM dragon.
Yes, we commissioned that in our hearts.
No, we will not be taking questions.

---

## the vibe, in a screenshot

<div align="center">
<img src="https://raw.githubusercontent.com/Scratchpiler/scratchpiler/main/resources/editor.png" alt="The Scratchpiler editor inside Scratch" width="860" />
</div>

That's a dark-mode IDE with a command palette, a live variables panel, go-to-definition, rename-symbol, find-and-replace across every sprite, and a linter.

It is running **on scratch.mit.edu.**
Next to the cat.
The cat can see it.
The cat is fine with it.

---

## the pipeline (a love story in six acts)

```
 your .sdsl, written like a dignified person
        │
        ▼
 lexer ─► parser ─► typechecker ─► linter        (judgment, in four flavors)
        │
        ▼
 irgen: AST ──► SLVM IR                          (the part where it gets serious)
        │
        ▼
 SLVM: verify ─► optimize ─► legalize            (the part where it gets unserious again)
        │
        ▼
 slc: IR ──► Scratch block maps
        │
        ▼
 injector: finds the live VM by crawling the React fiber tree
           (this is a real sentence we wrote in a README and meant)
        │
        ▼
 the cat moves 10 steps. a compiler did that.
```

Every compile goes through the IR.
The "direct compiler" that used to exist has been removed.
It had it coming.

---

## receipts

We do not do vibes-based correctness. We do **differential fuzzing**, which is vibes-based correctness with a lawyer.

> **10,400** randomly generated programs.
> One independent reference interpreter acting as the oracle.
>
> **SLVM:** 0 mismatches.
> **The old direct compiler:** 253 failures.

And the old one gave us this gem, which we can never unsee:

```
fib(10)   →   old compiler: 5       SLVM: 55
```

Off by a factor of eleven on *Fibonacci*.
We now have a fuzzer that finds this kind of thing *at night, while we sleep*, like a tiny judgmental raccoon.

---

## things our language can do that it frankly should not

**Pointers. In Scratch.**
`&[x]`, `*[p]`, `alloc(n)`, `free(p)`. There is a hidden first-fit allocator.
It lives in a hidden stage list called `__heap`.
We have built a linked list in a children's programming language and then *freed the nodes*, like responsible adults.

```
set [node] to alloc(2)
set *[node] to [i]
free([node])
```

**Inline assembly.**
`__asm__ volatile(...)` emits raw Scratch opcodes. There is also `unsafe`, which emits opcodes that *do not exist*, because we respect your autonomy:

```
__asm__ volatile unsafe(
    vm_trigger_segfault("SIGSEGV");
)
```

The VM shrugs. The VM always shrugs.

**Scratchroutines.**
Named concurrent tasks with `launch`, `await` and `cancel`. Threads, but in a place where a "thread" is a sprite doing a little dance.

**A fork bomb.**
It's in `examples/`. Scratch caps clones at 300. The VM still sweats.

**GOTO, via broadcasts.**
`spaghetti-goto.sdsl` counts to ten using **five event handlers**. It is a masterpiece. It is also a cry for help.

**A decompiler.**
It pulls blocks back *out* of the VM and turns them into text. Necromancy, but with indentation.

---

## SLVM, for the people who want to feel something

- **Structured regions, not a CFG**, because Scratch has no jumps and we refuse to write a relooper to apologize to a C-block.
- **SSA values** over Scratch variables-as-memory.
- `bool` and `val` types that match the **hexagon** and **round** reporter shapes. Our type system is literally a shape sorter.
- An **effect lattice**: `pure < read < write < yield`.
- Constant folding that follows scratch-vm's `Cast` rules, which means `"abc" + 1` is a *legal thought* here.
- `slopt` and `slc`. You read that right. `opt` and `llc`, but for a cat.

Our non-goals, quoted directly from the README:

> - Being fast enough to matter at LLVM scale. The largest Scratch project is smaller than one LLVM test file.
> - **Taking itself seriously.**

We are extremely good at the second one.

---

## get involved

1. Install Tampermonkey.
2. Open any project at `scratch.mit.edu/projects/*/editor`.
3. Press **Alt+M**.
4. Press **Shift+Shift** to open *Search Nowhere*. You will see.
5. Open an issue. Open a PR. Send us the cursed thing you compiled.

We especially want:

- 🧪 programs that make the fuzzer cry
- 🐛 bugs in the *old* compiler that SLVM already fixed, so we can feel superior
- 🧵 a better word than "legalize" for what we do to spilled values in recursion
- 🐱 any information about whether the cat knows

---

## faq

**Is this a joke?**
The README is. The compiler passed 10,400 random programs with zero mismatches. Both can be true.

**Why not just use blocks?**
Because we have wrists, and opinions, and a `git diff`.

**Why a whole IR for a block language?**
Because "Scratch has no registers and no stack, and the cost model is blocks executed per frame" is *exactly* the kind of target where a proper IR pays off, and also because we could.

**Does the cat approve?**
The cat has been moving 10 steps for longer than some of our contributors have been alive. The cat has no opinions. The cat is a professional.

---

<div align="center">

**scratchpiler**: write the code, skip the puzzle pieces.
**slvm**: because "compiler infrastructure for a cat that moves 10 steps" was *right there*.

<sub>no sprites were harmed. several were cloned past the 300 limit.</sub>

</div>
