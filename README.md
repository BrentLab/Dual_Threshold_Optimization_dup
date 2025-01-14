# Dual Threshold Optimization
[![Rust Tests](https://github.com/BrentLab/Dual_Threshold_Optimization/actions/workflows/tests.yml/badge.svg)](https://github.com/BrentLab/Dual_Threshold_Optimization/actions/workflows/tests.yml)
[![Rust Linting and Formatting](https://github.com/BrentLab/Dual_Threshold_Optimization/actions/workflows/linting.yml/badge.svg)](https://github.com/BrentLab/Dual_Threshold_Optimization/actions/workflows/linting.yml)
[![Crates.io Version](https://img.shields.io/crates/v/dual_threshold_optimization?cacheBust=1)](https://crates.io/crates/dual_threshold_optimization)
[![Documentation](https://docs.rs/dual_threshold_optimization/badge.svg)](https://docs.rs/dual_threshold_optimization)


This library provides a comprehensive toolkit for performing
[Dual Threshold Optimization](https://doi.org/10.1101/gr.259655.119) (DTO)
originally proposed by Kang et al.  

DTO compares two ranked lists of features (e.g. genes) to determine the rank
threshold for each list that minimizes the hypergeometric p-value of the overlap
of features. This method was originally applied to comparing the results of a
paired set of binding assay results and perturbation assay results for a given
transcription factor, but could be used with any two ranked lists of features. 

DTO provides the following statistics on the optimal threshold pair and overlap:
- An empirical p-value of the optimal overlap which is derived from a
  permutation-based null distribution.
- An estimate of the lower bound of the FDR based on the derivations detailed in the
  paper linked above.

Note: Starting with version 2.0.0, the method was fully re-implemented in Rust. For
the original implementation, which is the version used in the paper linked above,
see version 1.0.0, implemented by Yiming Kang, in the releases.

## Table of Contents
- [Installation](#installation)
    - [Using the cmd line](#using-the-cmd-line)
        - [Output](#output)
    - [Using the library](#using-the-library)
    - [Development](#developer-installation-and-usage)
- [Algorithmic Details](#algorithmic-details)
- [Troubleshooting](#troubleshooting)
- [Acknowledgements](#acknowledgements)

## Installation

### With Cargo

If you have [cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html),
the Rust package manager, installed, then you can install a binary into your `$PATH`. 
This is the recommended install method for most users:

```bash
cargo install dual_threshold_optimization
```

You can also install an MPI enabled version. This depends on an MPI installation, eg 
openMPI, on your system.

```bash
cargo install dual_threshold_optimization --features mpi
```

### From a release binary

Binaries for ubuntu (`default` and `mpi`), mac and windows (both only the default) are
available on the [release page](https://github.com/BrentLab/Dual_Threshold_Optimization/releases).

If you are on a Mac, for example, and you do not need MPI (most users), then you would
download the binary called `dual_threshold_optimization-macos-latest-default`.

After downloading to your computer, you will need to make this executable by entering

```bash
chmod +x dual_threshold_optimization-macos-latest-default
```

in your terminal. For windows, if you are not using the terminal, consult the internet
for the equivalent.

You may also want to rename the executable to something more manageable, e.g.

```bash
mv dual_threshold_optimization-macos-latest-default dual_threshold_optimization
```

to rename the executable to simply `dual_threshold_optimization`.


### Using the cmd line

With the correct binary, you can print the help message like so (omit the `./` if the 
binary is in your `$PATH` of course):

```bash
./dual_threshold_optimization --help
```

```bash
Dual Threshold Optimization CLI

Usage: dual_threshold_optimization [OPTIONS] --ranked-list1 <FILE> --ranked-list2 <FILE>

Options:
  -1, --ranked-list1 <FILE>
          Path to the first ranked feature list (CSV format).
          
          This should have two columns: "feature" and "rank". There should be **NO HEADER**.
          
          Rank is expected to be an integer. It is recommended that ties are handed with the `min` or `max` method.

  -2, --ranked-list2 <FILE>
          Path to the second ranked feature list (CSV format)
          
          This should have two columns: "feature" and "rank". There should be **NO HEADER**.
          
          Rank is expected to be an integer. It is recommended that ties are handed with the `min` or `max` method.

  -b, --background <FILE>
          Path to the background feature list (one feature per line, optional)

  -p, --permutations <PERMUTATIONS>
          Number of permutations to perform
          
          [default: 1000]

  -t, --threads <THREADS>
          Number of threads to use per task. For single-node parallelization, this will be the number of threads available on the machine.
          
          For multi-node parallelization, this will be the number of threads per task.
          
          Example: If you submit via Slurm with `-ntasks 4 --cpus-per-task 10`, then this value should be set to 10. This configuration will run 40 permutations in parallel.
          
          [default: 1]

  -m, --multi-node
          Enable multi-node mode using MPI. This requires that the program has been built with the `mpi` feature enabled

  -h, --help
          Print help (see a summary with '-h')

  -V, --version
          Print version

```

You can run this with the following minimal test data:
<!-- TODO: update the links when this goes to the main branch -->
- input list examples: [list1](https://raw.githubusercontent.com/BrentLab/Dual_Threshold_Optimization/refs/heads/main/test_data/ranklist1.csv), [list2](https://raw.githubusercontent.com/BrentLab/Dual_Threshold_Optimization/refs/heads/main/test_data/ranklist2.csv)
- background example: [background](https://raw.githubusercontent.com/BrentLab/Dual_Threshold_Optimization/refs/heads/main/test_data/background.txt)

like this (the background file is not required, but could be provided):

```bash
# download list1
wget https://raw.githubusercontent.com/BrentLab/Dual_Threshold_Optimization/refs/heads/main/test_data/ranklist1.csv 
# download list2
wget https://raw.githubusercontent.com/BrentLab/Dual_Threshold_Optimization/refs/heads/main/test_data/ranklist2.csv

# run the binary
dual_threshold_optimization -1 ranklist1.csv -2 ranklist2.csv -p 5 -t 1
```
This will output some run information to stderr, and a json to stdout. The json in the
stdout is the output of the program. This is important because it means that you can
re-direct the stdout to a file (see below) without saving the run metadata.

#### Output

The output is a json format string to stdout. To redirect this to a file, you would do
the following:

```bash

dual_threshold_optimization -1 ranklist1.csv -2 ranklist2.csv -p 5 -t 1
 > output.json
```

This is what the output will look like:

```json
{
  "empirical_pvalue": 0.8,
  "fdr": 0.0,
  "population_size": 30,
  "rank1": 24,
  "rank2": 13,
  "set1_len": 24,
  "set2_len": 13,
  "unpermuted_intersection_size": 12,
  "unpermuted_pvalue": 0.15632183908046102
}
```

Where the fields are the following:

- **empirical_pvalue**: The quantile of the unpermuted minimum p-value in relation to
  the series of permuted minimal p-values
- **fdr**: The lower bound of the false discovery rate where the sensitivity is set to
  0.8. See the [DTO paper](https://doi.org/10.1101/gr.259655.119) for more details
- **population_size**: The size of the background. If no background is explicity
  provided, this is the length of the input lists (when no background is provided, 
  the lists must contain the same set of features)
- **rank1**: The optimal rank for the unpermuted minimum p-value for list 1
- **rank2**: The optimal rank for the unpermuted minimum p-value for list 2
- **set1_len**: The number of features with rank less than or equal to **rank1** in
  list1
- **set2_len**: The number of features with rank less than or equal to **rank2** in
  list2
- **unpermuted_intersection_size**: The number of genes in the intersection of
  list1 and list2 with rank less than or equal to their respective optimal ranks
- **unpermuted p-value** the optimal p-value of the unpermuted lists. For analysis
  purposes, the empirical p-value should be used.

### Using the library

To use the library in your own Rust program, you can
`cargo add dual_threshold_optimization` in your rust project. See the crates.io
[documentation](https://docs.rs/dual_threshold_optimization/latest/dual_threshold_optimization/)
for more information about what is provided in each of the submodules.  

### Developer installation and usage

It is assumed that you have the
[rust toolchain](https://www.rust-lang.org/tools/install) already installed.

1. git clone this repository
2. `cd` into `Dual_Threshold_Optimization`

For any of the commands below, you can add `--features mpi` to include the MPI
feature. But, remember that this requires that MPI exist in your environment
(e.g. [openMPI](https://www.open-mpi.org/))

You can build an optimized binary with

```bash
cargo build --release
```

You can run the tests with:

```bash
cargo test
```

and you can run the debug binary with

```bash
cargo run -- --help
```

Note that there is a build profile for time and memory performance profiling which
will build a release version with the debug flags on:

```bash
cargo build --profile release-debug
```

### Test data

Minimal test data can be found in the
[test_data](https://github.com/BrentLab/Dual_Threshold_Optimization/tree/main/test_data) subdirectory

### Performance Profiling

I recommend profiling with [hyperfine](https://github.com/sharkdp/hyperfine)
for runtime and [heaptrack](https://github.com/KDE/heaptrack) for memory.
The results of profiling on the test data are in the `/profiling` subdirectory. Use the
`--profile release-debug` profile to build an executable for performance profiling.

### Pre-commit and CI

[Pre-commit](https://pre-commit.com/#install) is set up to run `cargo fmt` and
`clippy` when you commit changes. After pulling the repo, `cd` in and run
`pre-commit install`. There is also github actions CI set up to run the test suite,
the linters (`fmt` and `clippy`), and on pulls to `main`, to create a release. In
order for the release workflow to succeed, the version in `Cargo.toml` must not be the
same as the current state of `main`. The release CI will build the binaries and add
them to the release. You are responsible for updating the release notes after the
workflow completes.

## Algorithmic details

The following provides details on the DTO algorithm, step by step.

1. Initialize two ranked feature lists

    Begin with two ranked lists of features, e.g. genes, where each feature has
    an id, e.g. a unique identifier for the gene, and a rank. The rank must be an
    integer and is expected to have ties handled with a method such as "min" or "max"
    where ties all are assigned the same rank.
    
1. Create a series of thresholds for each list based on the ranks

    For each list, generate a series of thresholds T1, T2, ... . These thresholds are
    used to generate sets of features from each list to compare the overlap.
    The thresholds are calculated by the recurrence relation

    <p align="center">
      T<sub>1</sub> = 1<br/>
      T<sub>n</sub> = ⌊ T<sub>n-1</sub> * 1.01 + 1 ⌋
    </p>
    
    The stopping condition is when the threshold meets or exceeds the largest rank.
    The final threshold is always set to the max rank. This series provides finer
    spacing at higher ranks, allowing more granular selection among top-ranked genes.  

    The effect of this equation is that for the first 100 ranks, the thresholds
    increment at the same rate as the ranks, so we have 1, 2, 3, ... . At 100, the 
    resolution decreases by 2, eg 100, 102, 104, ... . For every additional 100
    ranks after this, the resolution decreases by 1, so for instance:
    200, 203, 206, ..., 402, 407, ..., 1705, 1723, 1741

1. Conduct a brute force search of the threshold pairs to find an optimal overlap

    For each possible pair of thresholds, select the genes from each list with rank
    less than or equal to the respective threshold. Calculate the hypergeometric
    p-value by intersecting the feature sets. This is the core of the algorithm with
    a complexity of O(n^2) where n is the length of the threshold lists.

1. Report the optimal threshold pair

    Return the threshold pair that describes the respective rank of each list that
    produces the feature sets that result in the minimum hypergeometric p-value
    (one-sided, upper only) across all tested threshold pairs. This threshold pair is
    considered optimal for identifying significant overlap between two ranked feature
    lists.

    **CAVEAT**: Though infrequent, due to the interplay between
    parameters of the hypergeometric distribution, it is possible that multiple sets
    yield the same p-value, including the minimal p-value. When this occurs on the
    minimal p-value, the threshold pair that yields the largest overlap is selected 
    as optimal. When there are multiple threshold pairs that have the same p-value and
    the same intersect size, the first in the set is chosen arbitrarily.

1. Use permutations to generate a null distribution for the minimal p-value

    To assess the statistical significance of the identified threshold pair, run steps
    3 and 4 multiple times (e.g., 1000 times) on randomized versions of the
    ranked lists (features assigned to ranks arbitrarily). This creates a null
    distribution of the minimal p-value and allows calculation of an empirical p-value
    of observing the previously identified optimal threshold pair by chance.

1. Calculate false discovery rate (FDR)

    In the [DTO paper](https://doi.org/10.1101/gr.259655.119), an FDR is derived. This
    FDR is estimated for the optimal threshold pair.


## Troubleshooting

If you are using the MPI binary, then you must have MPI in your environment. If you
do have MPI installed, but you get an error similar to the one below:

```bash
./dual_threshold_optimization: error while loading shared libraries: libmpi.so.40: cannot open shared object file: No such file or directory
```

Then you need to find where the `libmpi.so.40` file lives and add
it to your `LD_LIBRARY_PATH` manually. E.g.

```bash
export LD_LIBRARY_PATH=/ref/mblab/software/spack-0.22.2/opt/spack/linux-rocky9-x86_64/gcc-11.4.1/openmpi-5.0.3-vjscapwoywmullqs3lj2mmdf7vyge4rk/lib:$LD_LIBRARY_PATH
```

## Acknowledgements

DTO was originally implemented by [Yiming Kang](https://github.com/yiming-kang). See version 1.0.0 for that
implementation. That work was described in
[Dual threshold optimization and network inference reveal convergent evidence from TF binding locations and TF perturbation responses](https://doi.org/10.1101/gr.259655.119)
