# Word Design for DNA Computing on Surfaces

This project encodes a DNA word-design problem as a Boolean satisfiability
(SAT) problem, writes it in DIMACS CNF format, solves it with Glucose 4.2.1,
and translates a satisfying model back into a readable set of DNA words.

The implementation is contained in
[`Word Design for DNA Computing on Surfaces.ipynb`](<Word Design for DNA Computing on Surfaces.ipynb>).

## Problem

The program searches for a set `S` of words of length 8 over the alphabet
`{A, C, G, T}`. Given a non-negative target size `k`, it determines whether a
set containing exactly `k` words exists under these constraints:

1. Every word contains exactly four symbols from `{C, G}`.
2. Every two distinct words differ in at least four positions.
3. For every `x, y` in `S`, including `x = y`, the reverse of `x` and the
   Watson-Crick complement of `y` differ in at least four positions.

The Watson-Crick complement exchanges `A` with `T` and `C` with `G`. Word
differences are measured using Hamming distance.

The original optimization problem asks for the largest possible set. This
project solves its decision variant for a chosen exact size. Testing different
target sizes can be used to search for the largest satisfiable value.

## Requirements

- Windows with Windows Subsystem for Linux (WSL)
- Python 3.10 or newer
- Jupyter Notebook or JupyterLab
- [Glucose 4.2.1](https://github.com/audemard/glucose/releases/tag/4.2.1)
  compiled inside WSL

The Python implementation uses only standard-library modules. Jupyter is
required to open and execute the notebook.

## User guide

### Set up the solver

1. Install Python 3.10 or newer, Jupyter Notebook or JupyterLab, and WSL.
2. Download Glucose 4.2.1 inside the WSL distribution.
3. Install the Linux compiler, GNU Make, and zlib development files required
   by Glucose. For example, on Ubuntu these packages are available as
   `build-essential` and `zlib1g-dev`.
4. Compile Glucose according to the instructions included with its source
   code. Record the absolute Linux path to the resulting executable, such as
   `/home/USER/glucose/simp/glucose_release`.

The path passed to the notebook must be a Linux path visible inside WSL. A
Windows path such as `C:\glucose\glucose.exe` will not work.

### Configure an instance

The **Preparing an instance** cell near the end of the notebook contains the
user inputs:

```python
words = GenerateAllEligibleWords()
solverPath = "/home/USER/glucose/simp/glucose_release"
set_size = 1
```

| Input | Format and meaning |
| --- | --- |
| `solverPath` | A string containing the absolute WSL path to the compiled Glucose executable. |
| `set_size` | A non-negative integer giving the exact number of words required in the solution. |
| `words` | A list of candidate DNA-word strings. By default, the notebook generates all eligible words. |

With the default candidate generation, every word is eight characters long,
uses only `A`, `C`, `G`, and `T`, contains exactly four occurrences of `C` or
`G`, and satisfies the self reverse-complement constraint.

To solve a smaller custom instance, replace the generated list with a Python
list, for example:

```python
words = ["AAAACCCC", "AAAACCCG"]
```

Custom candidate lists are not validated automatically. Each entry must be an
eight-character string over `{A, C, G, T}` and should satisfy the individual
GC-content and self reverse-complement constraints. Duplicate entries should
not be used. The requested `set_size` should not exceed the number of candidate
words unless an unsatisfiable instance is intended.

`ALPHABET` and `WORD_LENGTH` are implementation settings rather than normal
user inputs. Changing `WORD_LENGTH` alone does not generalize the problem,
because the GC-count and distance requirements are fixed at four.

### Run the notebook

1. Open the project directory in PowerShell.
2. Start Jupyter:

   ```powershell
   jupyter notebook "Word Design for DNA Computing on Surfaces.ipynb"
   ```

3. Open the notebook and run its cells from top to bottom.
4. Edit `words`, `solverPath`, and `set_size` in the **Preparing an instance**
   cell.
5. Run the preparation cell and then the final cell. The notebook constructs
   the CNF formula, calls Glucose, and decodes the result.

After changing `words`, rerun the preparation cell so that the variable mapping
and compatibility clauses are rebuilt. After changing only `set_size`, rerun
the preparation and final cells to keep the displayed configuration and result
in sync.

### Interpret the output

For a satisfiable instance, the notebook prints the selected words, one per
line:

```text
Found solution for words:
AAAACCCC
```

This means that Glucose found exactly `set_size` mutually compatible words.

For an unsatisfiable instance, it prints:

```text
Glucose didnt find any solutions.
```

This means that no selection of exactly `set_size` words from the supplied
candidates satisfies all constraints.

Every run also creates or overwrites `glucoseClauses.txt` in the project
directory. This file contains the complete SAT instance in DIMACS CNF format:
the first line is the `p cnf` header and each remaining line is a clause ending
in `0`. The file can be hundreds of megabytes for the full candidate set.

The current implementation always writes the DIMACS file; it does not yet
provide a switch to suppress it. Glucose is run with verbosity disabled, so
solver statistics are not currently shown. These are known limitations with
respect to the project guideline requesting optional DIMACS output and solver
statistics.

### Troubleshooting

- If WSL reports that the solver cannot be found, verify that `solverPath` is
  an absolute Linux path and that the Glucose executable exists at that path.
- If `wsl.exe` or `wslpath` cannot be started, confirm that WSL and a Linux
  distribution are installed and working.
- If a custom word causes an index or complement error, check that it contains
  exactly eight characters and uses only `A`, `C`, `G`, and `T`.
- If the notebook uses an earlier configuration, rerun the preparation cell
  and all following cells.

## How it works

1. Generate all words of length 8 over the DNA alphabet.
2. Keep words that satisfy the GC-content and self reverse-complement rules.
3. Assign one Boolean variable to every eligible word.
4. Add a binary clause for every incompatible pair so that both words cannot
   be selected together.
5. Add auxiliary variables and clauses that require exactly `set_size` word
   variables to be true.
6. Write the formula in DIMACS format and run Glucose through WSL.
7. Decode the satisfying assignment back into DNA words.

The current implementation generates 17,440 eligible candidate words. The
notebook contains the detailed implementation and encoding documentation.

## Example instances

### Satisfiable

```python
words = ["AAAACCCC"]
set_size = 1
```

The expected result is **SAT**.

### Unsatisfiable

```python
words = ["AAAACCCC", "AAAACCCG"]
set_size = 2
```

Both words satisfy the individual-word constraints, but their Hamming
distance is 1, so they cannot both be selected. The expected result is
**UNSAT**.

## Limitations

- Solver execution currently depends on Windows and WSL.
- Input is configured by editing notebook variables; there is no command-line
  interface or external input format.
- The DIMACS file is always generated.
- Solver statistics are not currently displayed because Glucose runs with
  verbosity disabled.
- Large instances can require substantial time, memory, and disk space.

## Project files

| File | Description |
| --- | --- |
| `Word Design for DNA Computing on Surfaces.ipynb` | Implementation and detailed developer documentation. |
| `SPEC.txt` | Original problem specification. |
| `project rules and guidelines.txt` | SAT-project submission requirements. |
| `glucoseClauses.txt` | Generated DIMACS formula from the most recent run. |
