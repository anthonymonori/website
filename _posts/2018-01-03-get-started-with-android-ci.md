---
title: "Getting started with Continuous Integration on Android"
layout: post
date: 2018-01-03 23:00
headerImage: false
tag:
- guide
blog: true
star: false
author: anthonymonori
description: Work in progress, but feel free to peek in!
# description: Have a CI setup ready for your next open source project or get inspiration for your corporate setup at work! 
---

# WIP: This post is under construction. Feel free to read it and provide feedback via the [GitLab repo](https://www.github.com/anthonymonori/monori-me/issues) or on [Twitter](https://www.twitter.com/anthonymonori).

In the following blog post, I will try to cover the necessary steps on how to set up an CI pipeline for your next Android project!

Table of contents:
1. [Motivation](#1-motivation)
2. [Choosing a service](#2-choosing-a-service)
3. [Setup](#3-setup)
4. [Adding fastlane](#4-adding-fastlane)
5. [Adding danger](#5-adding-danger)
6. [What to do next](#6-what-to-do-next)

## 1. Motivation

Let's address elephant in the room first: _What is this CI you are talking about?_ Well, to quote the well-known [Martin Fowler](https://www.martinfowler.com/):

> Continuous Integration is a software development practice where members of a team integrate their work frequently, usually each person integrates at least daily - leading to multiple integrations per day. Each integration is verified by an automated build (including test) to detect integration errors as quickly as possible.

Sounds like something you want in your setup? Then you are in luck! Please carry on...

But before we go, let's address the _other_ elephant in the room: _What about Continuous Delivery? Isn't that the same?_ Well, again, I'll leave the explanation to the folks over at [ThoughtWorks](https://www.thoughtworks.com), who put this into words quite straight forward.

> Continuous Deployment is closely related to Continuous Integration and refers to the release into production of software that passes the automated tests.

Now with that out of the way, you may be having second thoughts as you may already do all this. A little bit of inspiration will always come handy, so stick around. **Also, the popular service Buddybuild.com got acquired by Apple and they announced that they are shutting down the support for Android builds by 1st of March, 2018.** This is a very bad news for Android folks who may already invested time to have their setup over there. Don't worry! I won't be covering buddybuild, but rather take the opportunity to give you suggestions on what to use instead! 

_As a little disclaimer: No service will last forever, so plan your next move carefully!_ 

## 2. Choosing a host/service

How does the landscape look as of January 2018? Well, some of the dominant players in this field are:

- [CircleCi](https://www.circleci.com)
- [Travis-ci](https://www.travis-ci.com)
- [Jenkins](https://www.jenkins-ci.org)
- [GitLab CI](https://docs.gitlab.com/ce/ci/) - if you already use GitLab
- [Bitrise.io](https://www.bitrise.io)
- [Visual Studio App Centre](https://appcenter.ms)

There may be more (and more upcoming ones), but as of today, these are the ones that dominate my Twitter feed.

While I can only recommend GitLab CI if you are already on the GitLab platform (CE or EE), it is a solution, where similar to Jenkins, you have to maintain your own build servers. This can be a useful solution if you want to have full control over it, but remember: _they need to be maintained, the knowledge and guides documented for your team, and scaling is not as easy as with an PaaS (Platform as a Service) solution. **If confidentiality is important for you, it may be the best shot for you!**.

As I will try to address hobbyists and open source contributors in this guide, I assume **confidentiality** is not your biggest concern, then let's look at some of the other solutions.

Unfortunately I can't speak of Bitrise.io and Visual Studio App Centre, as I have no hands-on experience with them. If you personally have any experience with either of the services, give me a shout [@anthonymonori](https://www.twitter.com/anthonymonori).

I personally use both Travis CI and CircleCI for personal projects. As of [Circle CI 2.0](https://circleci.com/docs/2.0/) they both operate in a containerized-environment, which allows some super-fast builds. While I find Travis CI okay from a performance perspective, I did had a hard time reading their _outdated_ documentation on [how to properly implement an Android configuration](https://docs.travis-ci.com/user/languages/android/) for their service. As an example, they do not mention how to utilize docker images for the ease of building, but they rather suggest that you use either one of their pre-installed _components_, or install your own packages.

As an example, here's a `.travis.yml` configuration I wrote some time ago based on their documentation:

```Yaml
language: android
jdk: oraclejdk8
sudo: false
before_cache:
  -rm -f $HOME/.gradle/caches/modules-2/modules-2.lock
  -rm -fr $HOME/.gradle/caches/*/plugin-resolution/
cache:
  directories:
    -$HOME/.gradle/caches/
    -$HOME/.gradle/wrapper/
env:
  global:
    - ADB_INSTALL_TIMEOUT=5

android:
  components:
    - tools
    - platform-tools
    - build-tools-26.0.2
    - android-24
    - android-22
    - extra-google-m2repository
    - sys-img-armeabi-v7a-android-22

before_install:
  - bash scripts/accept_licenses.sh
  - chmod +x gradlew

before_script:
  - echo no | android create avd --force -n test -t android-22 --abi armeabi-v7a
  - emulator -avd test -no-skin -no-audio -no-window &
  - android-wait-for-emulator
  - adb shell input keyevent 82 &

script:
  - ./gradlew clean test
  - ./gradlew assembleDebug
  - ./gradlew connectedDebugAndroidTest
```

_While you are free to copy it, I suggest you continue to read on and follow my setup guide for an even better solution!_

**To the contrary of the above solution** with Travis CI, I had only great experiences with CircleCI so far. It has a very **user friendly interface** and **great documentation** that tells you very easily how to leverage the containerized setup. 

### 2.1 Circle CI

Let's take a close look what CircleCI offers:

![](/assets/images/posts/get-started-with-android-ci/circleci-vs-buddybuild.jpeg)
_[Source](https://circleci.com/blog/migrating-from-buddybuild-to-circleci/?utm_content=bufferb2661&utm_medium=SCC&utm_source=Twitter&utm_campaign=Twitter-Buffer-Csy)_

Besides the above comperison, here are some of the other "perks" of using this product:

- Workflows for job orchestration
- Enterprise solution, besides the free pricing plan
- First-Class Docker support
- Choose the CPU/RAM you need
- Language-agnostic
- Powerful caching
- Insights

...and many others.

In the following section we will take a step by step approach on setting up Circle CI for your project on GitHub.

## 3. Setup

Head over to [www.circleci.com](https://circleci.com/) and login with your GitHub user. Once you authorize CircleCI to get access to your GitHub repositories (read-only), you are directed to the 'Projects >> Add Projects' page.

Here you will be prompted by two tabs and a list of your repositories. As we are focusing on building an Android CI pipeline for your Android apps, ensure you have the 'Linux' tab selected. This is due that we are going to orchestrate the pipeline on a Linux-based containerized setup. Then press 'Setup project' next to your desired project.

![](/assets/images/posts/get-started-with-android-ci/step-1-setup-project.png)

Once you have selected your project, keep the OS set to Linux and the Platform to `2.0`. As for the Language, just select 'Other' instead of Gradle (Java), as we will use our own configuration instead of the built-in.

![](/assets/images/posts/get-started-with-android-ci/step-2-configuration.png)

As the 'Next Steps' say, create a folder named `.circleci`, make a new file in that folder called `config.yml`.

![](https://media.giphy.com/media/l49JPsATsysQFTAxq/giphy.gif)

We will then paste the following configuration in there:

```Yaml
version: 2
jobs:
  build:
    working_directory: ~/code
    docker:
      - image: circleci/android:api-27-alpha
    environment:
      JVM_OPTS: -Xmx2400m
    steps:
      - checkout
      - restore_cache:
          key: jars-{ { checksum "build.gradle" } }-{ { checksum  "app/build.gradle" } }
      - run:
          name: Download Dependencies
          command: ./gradlew androidDependencies
      - save_cache:
          paths:
            - ~/.gradle
          key: jars-{ { checksum "build.gradle" } }-{ { checksum  "app/build.gradle" } }
      - run:
          name: Run Tests
          command: ./gradlew lint test
      - store_artifacts:
          path: app/build/reports
          destination: reports
      - store_test_results:
          path: app/build/test-results
```

Now let's disect a big this configuration before continuing

### 3.1 Docker image

As you can see in the 'build' job, we have a parameter called `docker` that defines the following image:

> circleci/android:api-27-alpha

CircleCi has their own docker images which they publish onto [Docker Hub](hub.docker.com) and you can find them [here](https://hub.docker.com/r/circleci/android/tags/). These are post-fixed by `-alpha` due to them being under constant development and CircleCi may or may not provide backwards compatibility in the future until they stabilize these images. They come with different api levels, and you should pick the one you use as your `targetSdkVersion`. If you use something below 23, you are doing something [bad](https://arstechnica.com/gadgets/2017/12/google-fights-fragmentation-new-android-features-to-be-forced-on-apps-in-2018/).

Alternatively, you can use my custom docker image that comes with Ruby, bbundler, node.js and yarn, and you can find that [here](https://hub.docker.com/r/anthonymonori/android-ci-image), but we'll come back to this later. I suggest you stick with the official images provided by CircleCi until further we get further in this post.

### 3.2 Steps

These are the steps your job will do while building. In our first example, we only have one job defined (called build), with multiple steps that do the following:

- checkout the source code with the given branch
- restore cache from a previous job that match the checksum of the build.gradle file
- download the required dependencies
- cache it for future use
- run lint and test
- store artifacts from the successful build
- store results from the lint and test step

This is it in short. You are ready to go and enjoy your fresh new CI pipeline, but if you stick around, I will go over some more advanced setup down below.

### 3.3 Secret variables

%TODO: _Coming soon_

### Workflows with multiple jobs

%TODO: _Coming soon_

## 4. Adding fastlane

%TODO: _Coming soon_

## 5. Adding danger

%TODO: _Coming soon_

## 6. What to do next?

%TODO: _Coming soon_
___

Thanks for sticking around and hope you enjoyed this (now long) tutorial! If you have any suggestions, or you spotted any issues, feel free to [submit a PR](https://github.com/anthonymonori/monori-me/pulls) or [file an issue](https://github.com/anthonymonori/monori-me/issues) to my blog on GitHub!