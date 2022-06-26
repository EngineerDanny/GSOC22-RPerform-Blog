---
toc: true
layout: post
description: This is the total review of how I went about in fixing the issues with dependencies in Rperform package.
badges: true
categories: [Rperform,performance, test, r, git, github]
title: Updating the dependencies in Rperform.
---


# Introduction
In R, it is the main purpose of the `DESCRIPTION` file to convey the metadata about a package. It is also needed to list all the dependencies that the package uses. Inside the `DESCRIPTION` file, there are `Imports` and `Suggests` section under which, you can list your dependencies. There is a minor difference between the two. The packages under `Imports` are needed by your users at runtime. `Packages` under `Suggests` however, may be optional because it is either used for development or optional functionalities. There was a major issue with the major dependencies used by `Rperform`. The issue was that, the versions of the dependencies used were too old and one of the them was discontinued.

## The git2r package
The `git2r` package is a popular R package which is mainly known for using git with R. It gives users programmatic experience with git using R. It is very fast because it implements git core methods developed with C. As the developers of the package improved and updated the package, they made a breaking transition. They made a change in support from S4 objects to S3 objects. A simple fix to this issue will be to pin the version of the package to specific versions in the `DESCRIPTION` file. However, a lot of developers have always install the latest versions of packages. As I tested `Rperform` for the first time, I had the issue. This is the error I had :

``` r
> plot_metrics(test_path = "tests/testthat/test-join.r", metric = "time", num_commits = 2, save_data = FALSE, save_plots = FALSE)
Error in base::substr(commit_val@summary, start = 1, stop = 15) : 
  trying to get slot "summary" from an object (class "git_commit") that is not an S4 object 
```

Analyzing the error message suggests that the program is trying to get a property from an object which is not S4 object but rather S3 object. 
The actual issue was that my `git2r` version was the latest version. For some reason, the package version which should have been pinned to be `<= version 0.21.0` was ignored by my program. The fact is that, the latest `git2r` package has a lot of new features and fixes. The first task I had to do was updating Rperform to also support the newer versions of `git2r` so as to make it easier for other developers to use and test the package. I updated the `git2r` version to `(>= 0.30.1)` in the `DESCRIPTION` file so it made some functions in the code obsolete. First of all, the names of some functions were updated. Refactoring the code was not difficult but running through the documentation of the `git2r` package and checking the newer functions proved to be a little challenging and time consuming. 
First major change I had to make was replace the `head` function to `repository_head` function. This function basically gets the `HEAD` of a repository.


``` r
# From
> repo_head <- git2r::head(repo)
```
``` r
# To
> repo_head <- git2r::repository_head(repo)
```

Also, I had to check all the functions related to the issue to be compatible to S3 objects. Just to name a few, some of them are :
``` r
# From
> base::substr(commit_val@summary, start = 1, stop = 15)
> repo_head@name
```

``` r
# To
> base::substr(commit_val$summary, start = 1, stop = 15)
> repo_head$name
```
Initially, I used numeric indexes to reference list members. I switched to using named reference approach to reference list members by names instead of numeric indexes because it was less error prone. With most of the changes just like above examples, I just had to replace the `@` symbol with the `$` symbol. Others were a little more complex than that.

## The Animint dependency
The [animint](https://github.com/tdhock/animint) package is a forked version of the `ggplot2` with improved features. It enables developers to design multi-plot, multi-layer, interactive and animated data visualizations. Unfortunately, the original version has been discontinued. Even though, it was an optional package in the root `DESCRIPTION` file, it was worth looking into. However, my attention only drew to it when I was running a `CMD R Check` on the `Rperform` package and it failed. The Github CI test showed that it is not possible to install it because it could not recognize the package. After digging into the issue, I realized the package has been discontinued. Fast forward, there is an updated version called the [animint2](https://github.com/tdhock/animint2) which comes with easier installation and better syntax. Therefore I updated the package to `animint2` in the `DESCRIPTION` file. Then, I checked and tested all the functions that depend on the animint2 package. Most of them had to do with just replacing `animint` with `animint2`.

``` r
# From
> animint::theme_animint(height = 700)
> animint::animint2dir(plot.list = viz.list, out.dir = paste0(basename(getwd()), "_", "time_animint"))
```

``` r
# To
> animint2::theme_animint(height = 700)
> animint2::animint2dir(plot.list = viz.list, out.dir = paste0(basename(getwd()), "_", "time_animint"))
```

## Fix dependency and related issues
After all the changes and updates, I opened a Pull Request to address the issues. [PR#2](https://github.com/EngineerDanny/Rperform/pull/2) and
[PR#3](https://github.com/EngineerDanny/Rperform/pull/3). As it was a pending issue raised twice on the main repository. These combined [PRs](https://github.com/analyticalmonk/Rperform/pull/47) close issues [#44](https://github.com/analyticalmonk/Rperform/issues/44) and [#45](https://github.com/analyticalmonk/Rperform/issues/45).

# Conclusion
I have realized that the dependencies that our projects utilize are very crucial to the development of the package and its users. At first, I nearly gave up on the `Rperform` package because of it's support for old dependencies which were not already installed and for some reason could not get installed  on my computer. Updating dependencies ensures the package gets access to the latest features, improvements and bug fixes. Downgrading the `git2r` dependency was a temporal fix, updating and ensuring that all related issues are fixed became the permanent fix.
