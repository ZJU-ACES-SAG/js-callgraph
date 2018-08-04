# Field-based Call Graph Construction for JavaScript #

This project implements a field-based call graph construction algorithm for JavaScript as described in

> A. Feldthaus, M. Schäfer, M. Sridharan, J. Dolby, F. Tip. Efficient Construction of Approximate Call Graphs for JavaScript IDE Services. In *ICSE*, 2013.


## Get Started
```
npm install
node src/main.js -h # for a list of command line arguments

# Running on simple input scripts
node src/main.js --cg input-scripts/simple-scripts/functioncall-arithmetic.js
```

## Running Tests
To run the testing framework run:
```
npm test
```
To install the git hooks to run tests automatically pre-commit run:
```
scripts/install-hooks
```
## Structure

The call graph constructor can be run in two basic modes (selected using the `--strategy` flag to `main.js`), *pessimistic* and *optimistic*, which differ in how interprocedural flows are handled. In the basic pessimistic approach (strategy `NONE`), interprocedural flow is not tracked at all; a slight refinement is strategy `ONESHOT`, where interprocedural flow is tracked only for one-shot closures that are invoked immediatel. The optimistic approach (strategy `DEMAND`) performs interprocedural propagation along edges that may ultimately end at a call site (and are thus interesting for call graph construction). Full interprocedural propagation (strategy `FULL`) is not implemented yet.

All strategies use the same intraprocedural flow graph, in which properties are only identified by name; thus, like-named properties of different objects are conflated; this can lead to imprecise call graphs. Dynamic property reads and writes are ignored, as are reflective calls using `call` and `apply`; thus, the call graphs are intrinsically incomplete.

Module `flowgraph.js` contains the code for extracting an intraprocedural flow graph from an [Esprima](esprima.org) AST (for convenience, a version of Esprima is included in `esprima.js`) annotated with name bindings for local variables (see `bindings.js`, which uses `symtab.js` and `astutil.js`).

Modules `pessimistic.js` and `semioptimistic.js` implement the pessimistic and optimistic call graph builders, respectively. They both use `flowgraph.js` to build an intraprocedural flow graph, and then add some edges corresponding to interprocedural flow. Both use module `callgraph.js` for extracting a call graph from a given flow graph, by collecting, for every call site, all functions that can flow into the callee position. Both use module `natives.js` to add flow edges modelling well-known standard library functions.

The remaining modules define key data structures, in several variants.

Module `graph.js` implements graphs using adjacency sets, using sets of numbers as implemented by `numset.js`. The latter includes either `olist.js` to implement sets as ordered lists of numbers, or `bitset.js` to use bitsets (with disappointing performance, so we use ordered lists by default).

Modules `dftc.js`, `heuristictc.js` and `nuutila.js` implement several transitive closure algorithms used by `callgraph.js`. By default, we use `dftc.js` which uses a simple, depth first-search based algorithm. `heuristictc.js` does something even simpler, which is very fast but unsound. Finally, `nuutila.js` implements Nuutila's algorithm for transitive closure, which for our graphs is usually slower than the depth first-based ones.


# License #

This code is licensed under the [Eclipse Public License (v2.0)](http://www.eclipse.org/legal/epl-2.0), a copy of which is included in this repository in file `LICENSE`.
