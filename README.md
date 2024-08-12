# Run GitLab jobs with updated syntax for deprecated only/except

## Motivation

Since `GitLab` version `13.0` the `only/except` syntax is deprecated as announced at: [https://about.gitlab.com/releases/2020/04/22/gitlab-12-10-released/#auto-devops-and-secure-configuration-templates-are-changing-to-rules-instead-of-onlyexcept](https://about.gitlab.com/releases/2020/04/22/gitlab-12-10-released/#auto-devops-and-secure-configuration-templates-are-changing-to-rules-instead-of-onlyexcept)
It is currently not known when it will be completely removed: [https://gitlab.com/gitlab-org/gitlab/-/issues/339180](https://gitlab.com/gitlab-org/gitlab/-/issues/339180)

It still is a good idea to switch to `rules` earlier rather than later.

Assuming you have a pipeline with 2 jobs, one that runs on master, and one that runs on branches, as shown here:
```sh
stages:
  - build

build-main:
  tags:
    - shell
  stage: build
  script:
    - echo "Hello World! from main branch"
  only:
    - main

build-branches:
  tags:
    - shell
  stage: build
  script:
    - echo "Hello World! from other branches"
  except:
    - main
```
how do you update it to get rid of the deprecated `only/except`?

## Prerequisites

A `GitLab` repository where you can run your pipeline.

## Changes

Switch to using rules instead of `only/except`. For example:
```sh
  only:
    - main
```
becomes
```sh
  rules:
    - if: '$CI_PIPELINE_SOURCE == "main"'
      when: on_success
    - when: never
```
while
```sh
  except:
    - main
```
becomes
```sh
  rules:
    - if: '$CI_PIPELINE_SOURCE == "main"'
      when: never
    - when: on_success
```

While this works in most cases, you will notice that the pipeline triggers twice if you have a merge request opened on your branch. This is a new unexpected logic that was added with the update. To get rid of the duplication you have to disable this feature. You can do this by adding:
```sh
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: never
    - when: always
```
at the beginning of the pipeline file.

So the updated pipeline is:
```sh
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: never
    - when: always

stages:
  - build

build-main:
  tags:
    - shell
  stage: build
  script:
    - echo "Hello World! from main branch"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "main"'
      when: on_success
    - when: never

build-branches:
  tags:
    - shell
  stage: build
  script:
    - echo "Hello World! from other branches"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "main"'
      when: never
    - when: on_success
```
