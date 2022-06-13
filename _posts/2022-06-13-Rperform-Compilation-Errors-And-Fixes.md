---
toc: true
layout: post
description: Fixes to some of the compilation errors of Rperform
badges: true
categories: [rperform, test, r]
title: Fixes to some of the compilation errors of Rperform
---

## ABSTRACT
The main goal of the RPerform library is to make performance tests on R packages easier with GitHub.
However, currently, the initial version of the RPerform library is outdated in the sense that you would 
need to improvise before you attempt to test RPerform.
This article explains all the potential errors, issues and bugs you might encounter and how to solve them before you run the package.

The test package that is used to test the `Rperform` library is the `stringr` (https://github.com/tdhock/stringr) package.
Installation of the library has been discussed in the previous articles.
Use the `library(Rperform)` command to load the library.
The following command initializes the library by setting the working directory.

```r
setwd(dir= "/Users/newuser/Projects/RProjects/stringr")
```

## DIRECTORY ISSUES
### **- Working directory error**
The first error which might be thrown is :
```r
Error in setwd(dir= "/Users/newuser/Projects/RProjects/stringe") :
 cannot change working directory
```
The `dir` argument must be set to the directory of the package or project you want to test.


### **- `test-join` directory error** 
There has been an update with the `stringr` repository. `test-join.r` is not in the `tests/testthat` directory. The new directory is  `inst/tests/test-join.r` . Also, all the files in `inst/tests/` directory must be taken note. Misrepresenting them will result in errors. Hence, this implies that running :
```r
plot_metrics(test_path = "tests/testthat/test-join.r", metric = "time", num_commits = 10, save_data = FALSE, save_plots = FALSE)
```
will throw the error :
```r
Error in file(con , "r") :
    cannot open file "tests/testthat/test-join.r" : No such file or directory    
```

Instead, run :
```r
plot_metrics(test_path = "inst/tests/test-join.r", metric = "time", num_commits = 10, save_data = FALSE, save_plots = FALSE)

```

## MISSING PACKAGES
These are the packages that `Rperform` depends on. Not having it pre-installed may prevent you from running the package. We proceed by discussing the relevant dependencies and how to install them.

`git2r` : The git2r package gives developers access to Git repositories from R. The package uses the `libgit2` library which is a C implementation of the Git core methods. Therefore it allows developers to write native speed custom Git applications in any language that supports C bindings.


To install the development version of git2r. You can use :
```r
install.packages("git2r")
```
 it's easiest to use the devtools package:
```r
# install.packages("devtools")
library(devtools)
install_github("ropensci/git2r")
```

`ggplot2` : The `ggplot2` is an R package used for plotting graphs. It provides useful commands that helps to create complex plots from data in a data frame. It enables quality plots to be created with minimal coding, tweaking and adjustments. The package is therefore necessary for `Rperform` to visualize performance metrics. It provides an interface for determining what variables to plot and how they are shown to the user. Organized data saves massive time when using the package to make figures.


```r
# The easiest way to get ggplot2 is to install the whole tidyverse:
install.packages("tidyverse")

# Alternatively, install just ggplot2:
install.packages("ggplot2")

# Or the development version from GitHub:
install.packages("devtools")
devtools::install_github("tidyverse/ggplot2")
```

## CONCLUSION
It is very crucial to adequately set up the package for testing.
Some of the errors have been discussed and solutions have also been proposed.
It is advisable to always follow the documentation for latest the updates.
