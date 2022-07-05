---
toc: true
layout: post
description: Testing and Debugging Rperform to ensure that R CMD Check runs successfully
badges: true
categories: [rperform, test, r]
title: R CMD Check on Rperform
---

## INTRODUCTION

When it comes to publishing an R package, it is very prerequisite and crucial for the package to pass `R CMD Check`. `R CMD Check` is basically a test performed on an R package to check code for common problems whilst suggesting ways of fixing them. This means that it is also great tool to even utilize during the whole development process of a package. `R CMD Checks` comprises of about 50 different tests which are explained [here](https://r-pkgs.org/r-cmd-check.html#check-checks)

## RUNNING `R CMD CHECK` LOCALLY

Running` R CMD Check` on `Rperform` is a simple task. During development, it can be run locally by using the famous devtools package.

```R
devtools::check()
```

Also, the R Studio IDE has in-built features which makes it easy to run R CMD Checks right inside the IDE. Under Build section, there is a check button which does that. Using various methods above is synonymous to running the following command on the R console:

```R
R CMD check Rperform
```

However, when developers need to share or collaborate on a project, the check comes in handy if it can be run online in a container for everyone to see the results. It is more challenging to set-up the checks to run online. Mostly, this approach is used when the need arises to run `R CMD check` every time changes are made in code. Since we use GitHub, it is recommended that we do that with [Github Actions](https://github.com/features/actions). There are others such as [Travis CI](https://www.travis-ci.com) and [Appveyor](https://www.appveyor.com/) but we will stick with Github Actions due to its simplicity and power. In addition to that, it implements modern technologies with vast functionalities and growing community.


It must be taken note that running `R CMD Check` on any package will return only three types of notices:

> **ERRORS** : These are issues which are very fatal and likely to cause the package or its dependencies to fail to install.

> **WARNINGS** : Warnings are statements that are made to caution the developer to take note of certain vital issues. These issues are unlikely to be detrimental to the package but it is very crucial to not ignore them.

> **NOTES** : Notes are not crucial issues. They are just mild problems which are meant for developers to take note of.

*R CMD Check passes successfully when none of the above notices are returned*

## `R CMD CHECK` USING GITHUB ACTIONS
[`GitHub Actions`](https://github.com/features/actions) is an open-source technology that makes it easy for automation of software workflows. It has exceptional Continuous Integration/ Continuous Deployment (CI/CD) tools which enables developers to build, test and deploy code right from their GitHub repositories. 

It is very exciting that the R community has a number of repositories of Github Actions which can be used to perform any variety of CI tasks for R projects with example workflows. In Rperform, we use the following actions sequencially :

1. [actions/checkout](https://github.com/actions/checkout) - Checks out your repository to the `$GITHUB_WORKSPACE` so that repository's workflow can utilize it.
2. [r-lib/actions/setup-r](https://github.com/r-lib/actions/tree/v2/setup-r) - Sets up an [R](https://r-project.org) environment for use in GitHub Actions by basically downloading and caching a version of R.
3. [r-lib/actions/setup-r-dependencies](https://github.com/r-lib/actions/tree/v2/setup-r-dependencies) - Downloads and installs all the packages declared in the `DESCRIPTION` file.
4. [r-lib/actions/check-r-package](https://github.com/r-lib/actions/tree/v2/check-r-package) - Runs `R CMD check` on an R package.

If every check passes without any discrepancies, it implies that the package is stable. However, if any of the check fails, detailed logs will be shown at the end of every action for you to rectify this issue. It can be very confusing at first when some of the checks fail but with a little bit of debugging and testing, you can get the package to pass.
When you check my workflow history on my Rperform repository [here](https://github.com/EngineerDanny/Rperform/actions), you would notice that I have about 50% failed workflow runs. I wrote an [article](https://engineerdanny.github.io/GSOC22-RPerform-Blog/rperform/performance/test/r/git/github/2022/06/20/Updating-dependencies-In-Rperform.html) describing a total review of how I went about fixing the issues with Rperform's initial dependencies. At that time, I had successive workflow failures (From` R-CMD-check #17` to` R-CMD-check #21` ). After fixing the various issues that popped up, I submitted a [Pull Request](https://github.com/EngineerDanny/Rperform/pull/1) explaining the series of failures and the steps I took to fix them. 

I will be diving deep into the [PR](https://github.com/EngineerDanny/Rperform/pull/1) to explain some of the issues encountered. 

<img alt="Screenshot 2022-06-22 at 11 44 48 AM" src="https://user-images.githubusercontent.com/47421661/175021313-4fcd05ff-fc67-45d1-8935-8e1ccf99feec.png">

From the [Check-RPerform #3](https://github.com/EngineerDanny/Rperform/actions/runs/2536152401), Github showed a visual evidence that points to `r-lib/actions/check-r-package` as the specific action that failed. 

Initially, the issue was invalid link in the `test-repo-metrics.R `file.
I found out that the [hadley/stringr](https://github.com/hadley/stringr) link does not exist.
Even though, when you open the link you will be redirected to the actual repository which is the [tidyverse/stringr](https://github.com/tidyverse/stringr).

My test results were inconsistent (Failing more times and occassionally passing) due to this issue.
Therefore, I changed the github link to [tidyverse/stringr](https://github.com/tidyverse/stringr)
After running Rperform check locally, even though the test didn't explicitly give me an error, it failed. The only warning was : 

>   checking Rd cross-references ... WARNING
>   Missing link or links in documentation object 'Rperform-package.Rd':
>     ‘[git2r:git2r-package]{git2r}’
>   
>   See section 'Cross-references' in the 'Writing R Extensions' manual.

I fixed the warning by inserting the [git2r](https://github.com/ropensci/git2r) official github link.


Although the issue with the link was fixed and **R CMD Check** on my local machine passed, the GitHub CI action still failed.
So I started to dig into the [tidyverse/stringr](https://github.com/tidyverse/stringr) package tests.

Inside the https://github.com/tidyverse/stringr/blob/main/tests/testthat/test-dup.r file, there are three tests which are ran internally. 

<img width="642" alt="Screenshot 2022-06-22 at 2 30 21 PM" src="https://user-images.githubusercontent.com/47421661/175054951-9392a05a-0ef8-4443-84f4-f06fa950b562.png">

As shown above, all the other tests passed except the `uses tidyverse recycling rules` test.
This led to my conclusion that, this may be an internal issue with the `tidyverse/stringr` package.

The first two tests however, could be used in checking Rperform.

## FINAL FIX
After several debugging and testing, I finally found the final error that was causing the **Rperform R CMD Check** test to fail on GitHub CI. It turns out that during my adventures and explorations, I mistakenly created a file( [tests/testthat/test-plot-metrics.R]) on the `master` branch. 

<img width="696" alt="Screenshot 2022-06-22 at 8 14 21 PM" src="https://user-images.githubusercontent.com/47421661/175130169-1d1dd3f7-2f9e-43ec-b7cc-b88de65599ec.png">

As recorded from the logs above, in the file, there is an issue with the `str_c`  function because it does not exist. Therefore, I removed this file since it's not needed. It was created for experimental purposes.

After fixing all the issues, all the checks passed ✅ with[`Check-RPerform #11`](https://github.com/EngineerDanny/Rperform/actions/runs/2544889102) workflow run.

## CONCLUSION
During my tests, I realized the cause of the inconsistencies in my workflow results. When running `R CMD Check` locally, I noticed that if your internet network is slow, because the dependencies and other stuffs can not be downloaded, the test will pass with no warnings or errors. This may be a flaw in the `R CMD Check` tool. It can lead to false positives. The main reason why I didn't notice the mistakenly created file locally was because I was on another branch (`test-pkg`) instead of the main branch (`master`). The file was rather created on the `master` branch.

`R CMD Checks` are very crucial to the success of a package especially when the package is to be published on [CRAN](https://cran.r-project.org). When a package passes the checks with no errors, warnings and notes, there would be no need for human intervention before the package is published on [CRAN](https://cran.r-project.org).
