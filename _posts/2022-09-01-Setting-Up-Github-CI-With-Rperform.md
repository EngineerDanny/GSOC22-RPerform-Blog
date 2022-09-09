---
toc: true
layout: post
description: How to set-up Rperform Github CI
badges: true
categories: [rperform, test, r]
title: Setting Up Github CI With Rperform
---

### Introduction

Rperform has great flexibility and usability. It internally supports automating the tests of your code on Github and creating comments to display the results. To set-up Rperform for your GitHub CI worflow, initialize Rperform with the command :

```r
Rperform:init_rperform()
```
- The above command creates the Github workflow directory, which is the `.github/workflow/` directory and populates it with the needed workflow files (`rperform-receive.yaml` and `rperform-comment.yaml`).

- It also creates the `rperform` directory and populates it with files which make test configuration and customisation easier. 

### Running the script

One of the most important files in the `rperform` directory is the `script.R` file, which is where the Rperform test functions should be written. The functions written in the file will be ran when the workflow is triggered. You can write multiple Rperform functions in the `script.R` file.

With only these two functions, you can run four different types of tests:
1. `Rperform::plot_metrics()` 
2. `Rperform::plot_branchmetrics()`

**NOTE:** The `save_plot` and `save_image` parameters must always be set to `TRUE` so that the outputs will be saved and displayed without any issues.

After this set-up, anytime there is a Pull Request in the repository, the workflow will be triggered and the tests will be run. The results will be commented out by Github Bot in the PR.

### Customising the PR comment
There are the `header.R` and `footer.R` files inside the `rperform` directory which you can edit to customize the PR comment. The `glue` package is used to format and interpolate the strings so as to join the different sections of the comment. It must be noted that since any comment text is rendered in Markdown format, you must use the markdown syntax to edit the comment. 

### Results
Below is an example of the auto comment that is generated when the workflow is triggered by a PR.

<img width="824" alt="Screenshot 2022-08-12 at 11 55 57 AM" src="https://user-images.githubusercontent.com/47421661/184349192-2231a797-2de9-4de5-bf34-07ce49ac90db.png">

### Testing Rperform on Popular R packages

**Rperform Test 1**
- Test on the [styler](https://github.com/r-lib/styler) package, main repository [here](https://github.com/EngineerDanny/styler).

- Inside the `script.R` file, the following changes were made. Basically, the tests paths were updated to run the time_metrics functions. Below is the image of the updated script.R file.

```R
## TEST 1
Rperform::plot_metrics(
  test_path = "tests/testthat/test-rmd.R",
  metric = "time", num_commits = 140,
  save_data = TRUE,
  save_plots = TRUE,
  total_width_in = 20
)

## TEST 2
Rperform::plot_metrics(
  test_path = "tests/testthat/test-escaping.R",
  metric = "time", num_commits = 140,
  save_data = TRUE,
  save_plots = TRUE,
  total_width_in = 20
)
```

- Sample [comment](https://github.com/EngineerDanny/styler/pull/1#issuecomment-1229242369) created by running the tests.
<img width="684" alt="Screenshot 2022-09-04 at 4 46 58 PM" src="https://user-images.githubusercontent.com/47421661/188338179-cc4f8371-29ff-4745-91e4-0a6282806b75.png">

**Rperform Test 2**
- Test on the [devtools](https://github.com/r-lib/devtools) package, original repository [here](https://github.com/r-lib/devtools).

- The `script.R` file is shown below :

```R
## TEST 1
Rperform::plot_metrics(
    test_path = "tests/testthat/test-utils.R",
    metric = "time", num_commits = 100,
    save_data = TRUE,
    save_plots = TRUE,
    total_width_in = 30
)

## TEST 2
Rperform::plot_metrics(
    test_path = "tests/testthat/test-check.R",
    metric = "memory", num_commits = 100,
    save_data = TRUE,
    save_plots = TRUE,
    total_width_in = 30
)
```

- Sample [comment](https://github.com/EngineerDanny/devtools/pull/1#issuecomment-1229734119) created by running the tests. 

<img width="914" alt="Screenshot 2022-09-04 at 7 23 14 PM" src="https://user-images.githubusercontent.com/47421661/188348860-1a94a080-eaf0-455d-a6d1-299617269df2.png">


