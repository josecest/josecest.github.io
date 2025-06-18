---
layout: page
title: C0 Compiler
description: with background image
img: assets/img/12.jpg
importance: 1
category: work
---

Developed as a semester-long project, this compiler interprets and generates assembly code for C0 (a language utilized by students in the introductory data structures and algorithms course). It was primarily written in Rust, taking advantage of many of language's features (structs, displays, etc.) in order to streamline the compiling process!

At its core, the compiler is divided into the following subsections:

Parser and Lexer: Given C0 code, the parser and lexer will tokenize the text and produce an AST, based on the definitions and grammar provided in c0.lalrpop. Currently, this lexer and parser support control flow, type definitions, function calls, and other basic operations.

Elaboration: After being provided the AST from the parser, this stage of the compiler elaborates the tree into a simpler form which is easier to process. Examples from this stage include transforming UnOps to BinOps (for example, ! into an equivalent ternary operator) and elaborating for loops into their equivalent while loop forms. This creates a nice AST for the later stages, preventing complex conversion from one form to another. One simple optimization that has been implemented in this stage is AST flattening, wherein representations containing nested sequences with nops are simplified to their most basic form. This helps greatly with the typechecker, as we avoid unnecessary recursion (avoiding stack overflows and increasing speed).

Typechecker: With the newly elaborated AST from the previous elaboration phase, the typechecker will traverse this tree and ensure that statements and expressions correspond to their correct type. It contains representations for functions, typedefs, and variables, checking for shadowing, function calls (and their type correctness), and nested typedefs. Moreover, the typechecker will also check that all control flow paths within the program will eventually return with the correct type on all of them. It uses a recursive algorithm, corresponding to the inference rules given to us (making it simpler to understand). We reduce statements and expressions until reaching a basic type (true, false, number, variable), and then go back up the tree while checking for general validity.

Forest IR: After typechecking has been completed, we move to another intermediate form: the forest. This intermediate representation is comprised of a series of commands, which begin to resemble a form of abstract assembly. Through processing the elaborated AST, we generate these commands, creating sequences to produce the values needed in our statements. Here, we make sure to push operations with side effects to the top of each expression and statement, forcing a runtime exception for the user as early as possible.

Code Generation: Given the forest IR, the compiler will run a maximal munch algorithm, where we've manually added special cases for modulus/divison operations and returning. We also include additional special casing for shifting operations, which must utilize the %cl register. Through this munching, we generate abstract assembly which then gets passed on to the next stage.

Control Flow: From our abstract assembly, we can now generate a control flow graph for our provided program. This is done through the creation of a basic block graph, where each basic block is defined as either having one entry point (a label) and one exit point (a return, goto, or if statement). By enforcing these invariants, can succintly represent our program as a group of interconnected blocks. The CFG contains information on the blocks and their associated instructions, along with a mapping of parents and children for each block.

Domination: Through our control flow graph, we are now able to calculate the dominator tree and dominance frontier for our program. These structures allow us to reason about variables and their definitions, along with determining where to insert phi functions for SSA. Our implementation is modeled after the one described in "A Simple, Fast Dominance Algorithm"

Phi Insertion: With the information from our control flow graph and domination algorithms, we follow the algorithm found in the SSA book for phi insertion. These phi functions intuitively represent locations where a variable can be defined from multiple places. Through the dominance frontier, we can determine the locations where phi functions may need to be inserted for variables. This part of the compiler outputs an updated control flow graph with the phi functions inserted into the appropriate blocks. It is important to note that these phi functions are currently blank, and will have the appropriate variables inserted during variable renaming.

Variable Renaming: Variable renaming uses the control flow graph generated after phi insertion to perform variable renaming and bring the program into SSA. Our implementation models the one found in the SSA book, using reaching definitions to update the phi functions and insert the proper variable definition within them. It will also update the destination and source variables for every block, in order to reflect their most recently dominating definition. This stage of the compiler outputs a control flow graph in SSA form!

Deconstruct SSA: Deconstruct SSA uses the concept of phi webs from the SSA book to replace variables represented by phi functions with real temps. Additionally, it converts the control flow graph back into abstract assembly.

Liveout Analysis: Once the compiler has its set of abstract assembly lines, it begins to calculate the live-out and live-in sets for each temp/variable declared. This involves traversing all lines in the abstract assembly and updating live-in and live-out sets as needed. Once no changes are detected, we collect this data and forward it to the register allocator. This part was likely the most annoying with regard to rewriting, as for L3 it had to be completely rewritten to support function calls and their interactions with control flow.

Register Allocation: The register allocation stage takes in the live-in/live-out sets and produces an interference graph (which is done using Rust's built-in HashMap structure). With this, it performs SEO (simplical elimination ordering) in order to then perform greedy coloring. This generates a mapping between temp and register that is used in the next stage.

Instruction Selection: With our abstract assembly and register allocation, the compiler is finally able to produce x86-64 assembly. At the moment, we go through each abstract assembly instruction, translating the temps to their particular registers and creating instructions that are passed onto an "instruction list". This varies between instructions, as some like div/mod require special handling due to register limitations. Apart from that, our general algorithm works on all cases.

x86-64 Production: Once we have our instruction list, we print them out onto an assembly file utilizing Rust's unique "Display" feature. This finally leaves us with our assembly generated!


To give your project a background in the portfolio page, just add the img tag to the front matter like so:

    ---
    layout: page
    title: project
    description: a project with a background image
    img: /assets/img/12.jpg
    ---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images.
Say you wanted to write a bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, _bled_ for your project, and then... you reveal its glory in the next row of images.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}

```html
<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
```

{% endraw %}
