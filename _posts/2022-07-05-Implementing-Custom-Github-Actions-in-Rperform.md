---
toc: true
layout: post
description: My step-by-step approach to develop the prototype of custom Github Actions in Rperform. 
badges: true
categories: [rperform, test, r]
title: Implementing Custom Github Action Prototype in Rperform
---

## ABSTRACT
In my previous [article](), I established the reason why Github Actions was the preferred choice over other tools like [Travis CI](https://www.travis-ci.com) and [Appveyor](https://www.appveyor.com/). The main goal of this project as described [here](https://github.com/rstats-gsoc/gsoc2022/wiki/RPerform) is to get Rperform working and easy to use with `Github Actions`. GitHub Actions enable DevOps to be done on Github and allows workflows to be run when other events are triggered in the repository. GitHub provides macOS, Linux and Windows virtual machines to run workflows. A workflow is a configurable process defined by YAML file which contains one or more jobs that may be run sequencially or parallel. Each job runs inside it's own container or runner. Under each job, there are steps that runs a script or an action. 

## DEVELOPMENT CONSIDERATIONS
### The repository
To develop a custom Github Actions, I had to consider whether to keep it in the main repository or create a new repository just for the Github Actions. I decided to keep it in the main repository because it is easier to maintain along with the main package as well. The main reason why I chose to do that was because this particular Github Action is specific to only this package and it is not generalized enough to be used with other packages. 

The only issue that I thought of was version control of the action. However, it can be fixed by specifying the version number of the action which must be the same as the version number of the Rperform package.
e.g. `Rperform v0.0.1`:
```yaml
- uses EngineerDanny/Rperform/actions/receive@v0.0.1
```


### The Type of Action
According to the [documentation](https://docs.github.com/en/actions/creating-actions/about-custom-actions#types-of-actions), there are only three types of actions namely :
- Docker Container which supports only Linux OS
- Javascript which supports Linux, MacOS and Windows 
- **Composite Actions** which also support Linux, MacOS and Windows.

Even though all the other types of actions are very important, I chose to use composite actions. It allows the combination of multiple workflow steps within one action. This feature makes it possible to combine multiple run commands into an action. If a workflow is set-up, it can execute the commands as a single step with that action. This flexibility is the main reason why I adopted this approach. 

## IMPLEMENTING CUSTOM GITHUB ACTIONS PROTOTYPE

I submitted a [PR](https://github.com/EngineerDanny/Rperform/pull/7) which intends to create Github Actions to make it easier for package developers to use Rperform to test their code. The Actions are tested on a separate repository [stringr](https://github.com/EngineerDanny/stringr/pull/2).

To make set-up easier, the Github workflow directory, which is the `.github/workflow/` directory, is created automatically when you ran 
`Rperform::init_rperform()` function. It is also populated with the needed workflow files to be able to run `Rperform` with ease. It adds the `rperform` directory and populates it with files which make test configuration and customisation easier. One of the most important files in that directory is the `script.R` file which is where the Rperform test functions should be written.
<img width="485" alt="Screenshot 2022-07-01 at 3 33 40 PM" src="https://user-images.githubusercontent.com/47421661/176925448-eb5237f5-33ad-40e6-94a9-4c8dbaa81744.png">


When writing the Github Action for Rperform, I divided the action into two parts. These are the **`receive`** and **`comment`** parts. This decision was taken due to security reasons. 

### [The Receive Action](https://github.com/EngineerDanny/Rperform/tree/main/actions/receive)

* This action is represented by [EngineerDanny/Rperform/actions/receive](https://github.com/EngineerDanny/Rperform/tree/main/actions/receive)
  * Triggered via PR or push to main branch.
  * Reads `config.json` to prepare and run the benchmark job.
  * Does not have read or write access due to [security reasons](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/).

* Below is the script inside receive.yaml file

```yml
name: "receive"
description: "Action to run Rperform benchmarks and upload the results."
inputs:
  cache-version:
    description: "Integer to use as cache version. Increment to use new cache."
    required: true
    default: 1
  rperform_ref:
    description: "Which branch or tag of Rperform should be used. Mainly for debugging."
    required: true
    default: "@github-actions"

runs:
  using: "composite"
  steps:
    - name: Checkout repo
      uses: actions/checkout@v3
      with:
        fetch-depth: 0
    - name: Set up git user
      run: |
        git config --local user.name "GitHub Actions"
        git config --local user.email "actions@github.com"
      shell: bash
    - name: Setup R
      uses: r-lib/actions/setup-r@v2
    - name: Install dependencies
      uses: r-lib/actions/setup-r-dependencies@v2
      with:
        cache-version: ${{ inputs.cache_version }}
        extra-packages: |
          any::ggplot2
          any::dplyr
          any::gert
          any::glue
          github::EngineerDanny/Rperform${{ inputs.rperform_ref}}
    - name: Remove global installation
      run: |
        pkg <- unlist(read.dcf('DESCRIPTION')[, 'Package'])
        if (pkg %in% rownames(installed.packages())) {
          remove.packages(pkg)
          cat('removed package ', pkg, '.', sep = "")
        }
      shell: Rscript {0}
    - name: Run benchmarks
      run: Rperform::run_script("rperform/script.R")
      shell: Rscript {0}
    - name: Uploading Results
      run: |
        echo "Uploading results..."
      shell: bash

    - name: Saves the PR number in an artifact
      shell: bash
      env:
        PULL_REQUEST_NUMBER: ${{ github.event.number }}
      run: echo $PULL_REQUEST_NUMBER > ./rperform/pr-comment/PR_NO

    - uses: actions/upload-artifact@v2
      with:
        name: pr
        path: rperform/pr-comment/
```


### [The Comment Action](https://github.com/EngineerDanny/Rperform/tree/main/actions/comment)

* This action is represented by [EngineerDanny/Rperform/actions/comment](https://github.com/EngineerDanny/Rperform/tree/main/actions/comment)
  * Comments the results on the PR that originated the workflow run. 
  * Starts right after the `receive` job finishes.
  * Will create an additional commit status to the PR check suite. 
  * Has read & write access.

* Below is the script inside comment.yaml file

```yaml
name: "comment"
description: "Action to comment Rperform results on the appropriate PR. Needs read/write access."
inputs:
  GITHUB_TOKEN:
    description: "The GITHUB_TOKEN secret."
    required: true

runs:
  using: "composite"
  steps:
    - name: "Download artifact"
      id: "download"
      uses: actions/github-script@v3.1.0
      with:
        script: |
          var artifacts = await github.actions.listWorkflowRunArtifacts({
             owner: context.repo.owner,
             repo: context.repo.repo,
             run_id: ${{github.event.workflow_run.id }},
          });
          var matchArtifact = artifacts.data.artifacts.filter((artifact) => {
            return artifact.name == "pr"
          })[0];
          var download = await github.actions.downloadArtifact({
             owner: context.repo.owner,
             repo: context.repo.repo,
             artifact_id: matchArtifact.id,
             archive_format: 'zip',
          });
          var fs = require('fs');
          fs.writeFileSync('${{github.workspace}}/pr.zip', Buffer.from(download.data));
    - run: unzip pr.zip
      shell: bash
    - name: "Comment on PR"
      id: "comment"
      uses: actions/github-script@v6
      with:
        script: |
          var fs = require('fs');
          var pr_number = Number(fs.readFileSync('./PR_NO'));
          var body = fs.readFileSync('./results.txt').toString();
          await github.rest.issues.createComment({
            owner: context.repo.owner,
            repo: context.repo.repo,
            issue_number: pr_number,
            body: body
          });
          
    - uses: actions/github-script@v5
      if: always()
      with:
        script: |
          let url = '${{ github.event.workflow_run.html_url }}'
          let any_failed = ${{ steps.comment.outcome == 'failure' || steps.download.outcome == 'failure' }}
          let state = 'success'
          let description = 'Commenting succeeded!'

          if(${{ github.event.workflow_run.conclusion == 'failure'}} || any_failed) {
            state = 'failure'
            description = 'Commenting failed!'
            if(any_failed) {
              url = "https://github.com/${{github.repository}}/actions/runs/" + 
                    "${{ github.run_id }}"
            }
          }
          github.rest.repos.createCommitStatus({
            owner: context.repo.owner,
            repo: context.repo.repo,
            sha: '${{ github.event.workflow_run.head_sha}}',
            state: state,
            target_url: url,
            description: description,
            context: 'rperform comment'
          })
```


## CONCLUSION
This is the prototype of the Github Actions. Initially due to separation of actions, I had an issue with the Github create comment script which was similar to [this issue](https://github.com/actions/github-script/issues/77). After several debugging and testing, I was able to successfully automate the creation of comments. [(Auto Comment On PR)](https://github.com/EngineerDanny/stringr/pull/2#issuecomment-1172716652). The actions are implemented in such a way that testing results are firstly uploaded as an artifact on Github. Then afterwards, a comment is created by Github actions bot on the PR. At this point, I only tested it with the `plot_metrics` function. More functionalities will be added to the actions in time. I am thinking about limiting the number of functions that can be run in the `script.R` file. 
