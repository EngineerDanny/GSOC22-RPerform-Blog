---
toc: true
layout: post
description: An in-depth review of touchstone, an R tool for benchmarking performance of code
badges: true
categories: [touchstone, test, r, git, github]
title: Reviewing The Touchstone R Package
---

## 1. Introduction
[`Touchstone`](https://github.com/lorenzwalthert/touchstone) is an R tool which provides accurate benchmarking features for testing other R packages. It provides continuous benchmarking with reliable relative measurement and uncertainty reporting.
**It is enriched with features which are very useful especially with respect to merging a Pull Request into target branch.** Therefore, it is integrated with GitHub Continuous Integration(CI) which helps to automate the whole process. 

## 2. Concept
For your PR branch and the target branch, [`touchstone`](https://github.com/lorenzwalthert/touchstone) will : <br>
- build two versions of the package in isolated libraries.
- measure the accurate relative differences between the branches. The code under experimentation will be run several times in a random order.
- comment the results of the benchmarking on the Pull Request.
- create visualizations to demonstrate the distribution of the timings for both branches.

## 3. Installation
Installation can be done in two ways : <br>
* **CRAN** :
``` r
install.packages("touchstone")
```
* **GitHub** : 
 ``` r 
 devtools::install_github("lorenzwalthert/touchstone")
 ``` 

## 4. Package Usage
Initialize touchstone by running <br> 
``` r 
touchstone::use_touchstone()
```

The above line of code will :
- Create a directory known as the `touchstone` directory in the root of the repository with `config.json` and `script.R`. The `config.json` contains configurations that define how to run your benchmark. The `script.R` is the script that runs the benchmark. The `header.R` contains the default PR header whilst the `footer.R` containing the default PR comment footer. 

- Populate the file `touchstone-receive.yaml` in `.github/workflows/.`
and `touchstone-comment.yaml` in `.github/workflows/` respectively.

- Add the `touchstone` directory to `.Rbuildignore` file in the root directory of your repository.

Write the workflow files  you need to invoke `touchstone` in new pull requests into `.github/workflows/`.

Modify the `touchstone` script `touchstone/script.R` to run different benchmarks.

## 5. Understanding the script
There are three default functions inside the `script.R` file.

### (i) `branch_install`
``` r
touchstone::branch_install()
```
`branch_install` installs each `branch` in a separate library for isolation.
#### Usage
``` r
branch_install(
  branches = c(branch_get_or_fail("GITHUB_BASE_REF"),
    branch_get_or_fail("GITHUB_HEAD_REF")),
  path_pkg = ".",
  install_dependencies = FALSE
)
```
**The arguments :** <br>
`branches` are names of the branches in character vector <br>
`path_pkg` represents the path to the package which has a default value of `"."` <br>
`install_dependencies` is a boolean which enables dependencies to be installed, has a default value of `FALSE` <br>

### (ii) `benchmark_run`
``` r
touchstone::benchmark_run()
```
`benchmark_run` runs benchmarks for git branches using function calls from your package.
#### Usage
``` r
benchmark_run(
  expr_before_benchmark = { },
  ...,
  branches = c(branch_get_or_fail("GITHUB_BASE_REF"),
    branch_get_or_fail("GITHUB_HEAD_REF")),
  n = 100,
  path_pkg = "."
)
```
**The arguments :** <br>
`expr_before_benchmark` allows an expression to be executed just before benchmark is run<br>
`...` is the named expression of length one with code to benchmark<br>
`branches` are names of the branches in character vector <br>
`n` is the number of times benchmarks should be run for each of the branches. <br>
`path_pkg` represents the path to the package <br>

### (ii) `benchmark_analyze`
``` r
touchstone::benchmark_analyze()
```
`benchmark_analyze` creates artifacts used downstream in the GitHub Action to turn raw benchmark results into text and figures.

#### Usage
``` r
benchmark_analyze(
  branches = c(branch_get_or_fail("GITHUB_BASE_REF"),
    branch_get_or_fail("GITHUB_HEAD_REF")),
  names = NULL,
  ci = 0.95
)
```
**The arguments :** <br>
`branches` are the names of the branches in character vector under consideration <br>
`names` are the names of the branches which is actually used for the analysis. <br>
`ci` represents the confidence level which defaults to 0.95 out of 1 <br>

## 6. Running the  script
On the R Interactive console, run
``` r
touchstone::run_script()
```
#### Usage
``` r
run_script(
  path = "touchstone/script.R",
  branch = branch_get_or_fail("GITHUB_HEAD_REF")
)

```
**The arguments :** <br>
`path` is the path to the script to run <br>
`branch` is the name of the branch corresponding to the library <br>
<br>

After committing and pushing the workflow files to default branch, `Github CI` will run the benchmarks on every pull request and on each commit while that pull request is open.
