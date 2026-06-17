# SMARtS — Strategy-based Maude Analyzer for Reaction Systems

SMARtS is a Maude-based framework for formalizing and analyzing **Reaction Systems (RS)**, a computational model originally introduced to capture the way biochemical reactions process and combine sets of molecular entities. SMARtS provides:

- An **executable formalization** of RS semantics in Maude (entities, reactions, reaction systems, and the step-by-step computation driven by an external context/environment).
- A **library of reusable Maude strategies** that control how a reaction system is executed — e.g. running an exact number of steps with a fixed context, running until a steady state is reached, or constraining the context to include/exclude specific entities.

The goal of SMARtS is to let users both *simulate* reaction systems under controlled strategies (using Maude's `srew` command) and *formally verify* temporal properties of their behavior (using `srew` as well as LTL/CTL model checking through the unified model checking tool `umaudemc`), all from the same underlying Maude specification.
More information can be found in our [technical paper](https://github.com/DemisGIT/SMARtS-tool/blob/main/SMARtS.pdf).

## Project structure

```
SMARtS-tool/
└── src/
    ├── RS-semantics.maude   --- Core RS data structures and rewriting semantics
    ├── RS-strategy.maude    --- Parameterized strategy library 
    └── experiments/         --- Ready-to-run experiments
```

## Experiments

The `src/experiments` folder contains short- as well as long-term experiments related to the
biological case study of [`[1]`](#related-papers), which analyses protein signalling networks in three HER2-positive breast cancer subtypes (namely, BT474, HCC1954, SKBR3) under different combinations of monoclonal antibody drugs. For each experiment the folder follows a consistent naming convention, illustrated here for BT474:

| File | Role |
|---|---|
| `BT474.maude` | Defines the reaction system itself: the entities, the `BT474` reactions, the environment for the short term analysis of BT474.  |
| `BT474-lt.maude` | Same as above, but for the long-term variant of the network (`BT474-LT`). |
| `BT474-exp.maude` | A **Maude script** that runs a battery of `srew` (strategy-controlled rewrite) commands exploring the system's behavior under different combinations of stimuli (e.g., activation of metabolic pathways, drug treatments). Also it includes an example of a "perfect-recall" strategy applied to the considered biological context. |
| `BT474-lt-exp.maude` | The analogous battery of `srew` experiments for the long-term variant. |
| `BT474-umaudemc` | A list of `umaudemc check` invocations (one per line) verifying LTL/CTL properties  over the same stimulus scenarios used in `BT474-exp.maude`.|
| `BT474-lt-umaudemc` | The analogous `umaudemc check` battery for the long-term variant. |

The same six-file pattern is repeated for `HCC1954` and `SKBR3`. 

## Running the experiments

All commands below assume you run them from the `src/examples` directory (so that the relative `load` statements resolve correctly), and that `maude` and `umaudemc` are available on your `PATH`.

### 1. Running Maude Reachability Analysis (`srew` experiments)

Files like `*-exp.maude`, `*-lt-exp.maude` are plain Maude files: running them with `maude` executes every `srew` command in sequence.

```bash
cd src/examples
maude BT474-exp.maude
```

You can also load any of these files from an interactive Maude session and issue further `srew` commands of your own:

```text
Maude> load BT474-exp
Maude> srew < BT474 | empty | nil > using se-sets(10,(egf e, egf p, egf t)) .
```

### 2. Model checking with `umaudemc`

Files like `*-umaudemc`, `*-lt-umaudemc` contain one `umaudemc check` shell command per line. Each line has the form:

```bash
umaudemc check <model-file> "<initial-state>" "<temporal formula>" "<strategy>"
```

You can run any single check directly, for example:

```bash
cd src/examples
umaudemc check BT474.maude "< BT474 | empty | nil >" "E <> ( akt /\ steady )" "s-without(1,e p t) ; se-steady(empty)"
```

Since these files only contain valid shell commands, you can also run an entire batch of checks at once:

```bash
cd src/examples
bash BT474-umaudemc
```

This will sequentially verify all the properties defined for the BT474 short-term model across every stimulus scenario.

## Related Papers

[1] der Heyde, S.V., Bender, C., Henjes, F., Sonntag, J., Korf, U., Beißbarth, T.: Boolean ErbB network reconstructions and perturbation simulations reveal individual drug response in different breast cancer cell lines. BMC Systems Biology