---
layout: post
title: 'Where should Gradle build logic live?'
date: 2026-08-26 23:11
tag: [blog, android]
categories: [android]
author: Antal Monori
description: Learn where Gradle build logic should live, from duplicated scripts and buildSrc to included builds, convention plugins, and a custom DSL for your build logic.
image: /assets/posts/2026-08-22-where-should-gradle-build-logic-live/header.png
---

As a project grows, its build logic tends to grow with it. What started as a short `build.gradle(.kts)` file becomes hundreds of scripts applying the same plugins, configuring the same toolchains, and declaring the same dependencies. Eventually, making a seemingly small change requires touching a large part of the project. This is where build logic starts deserving the same attention as any other part of your codebase.

In this post, we will start with duplicated Android build scripts and gradually move that logic into precompiled scripts, an included build, convention plugins, and finally a small custom Gradle DSL.

## What is build logic?

Your build contains the description of the software you want to produce:

- the modules that make up the application;
- the dependencies between those modules;
- the capabilities an individual module requires;
- and the variants or artefacts it should produce.

Build logic is the code responsible for turning that description into something Gradle can execute. It applies plugins, integrates tools, configures tasks, and establishes conventions across the project.

Here is an example build script for a typical Android module:

```kotlin
plugins {
    id("com.android.library")
    id("org.jetbrains.kotlin.plugin.compose")
}

android {
    compileSdk = 37
    defaultConfig { minSdk = 29 }
    buildFeatures.compose = true
}

dependencies {
  implementation(platform("androidx.compose:compose-bom:<version>"))
  implementation("androidx.compose.ui:ui")
}
```

There is nothing particularly wrong with this build script. It is explicit, readable, and easy to change. The problems start when we create another module. Or fifty.

At this point, the build starts experiencing some familiar growing pains:

- duplicated configuration across build scripts;
- inconsistent SDK or language versions;
- misconfiguration going unnoticed;
- a larger surface area to maintain;
- and slow, repetitive migrations.

Changing the `compileSdk` version should be a single decision. It should not become a pull request modifying every module in the repository.

## Why not configure every project from the root?

A common first thought we may have is to move the shared configuration into the root build script via the `subprojects {}` or `allprojects {}` constructs, [Gradle’s own documentation](https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html#sec:convention_plugins_vs_cross_configuration) recommends against this! It becomes hard to understand which configuration applies to which projects, and quickly can become a burden. It also works against Gradle features designed to configure projects more independently.

## Creating conventions to share build logic

### Precompiled scripts

A [precompiled script plugin](https://docs.gradle.org/current/userguide/implementing_gradle_plugins_precompiled.html) is a `.gradle` (Groovy DSL) or `.gradle.kts` (Kotlin DSL) script stored in a plugin source set. Gradle compiles it into a plugin and derives its plugin ID from the filename. For the remainder of this post, we will use Kotlin DSL because it is familiar to many Android engineers.

You can get started by creating a top-level `buildSrc` project at the root of your build:

```
buildSrc/
├── build.gradle.kts
└── src/main/kotlin/
    └── sample.android.library.gradle.kts
```

The name of the file exposes the plugin as `sample.android.library`, which can then can already be applied to your project build scripts using `plugins { id("sample.android.library") }`

See this in [[Step 1] Extract build logic into buildSrc /w precompiled scripts](https://github.com/anthonymonori/sample-gradle-project/pull/5).

## Is `buildSrc` the right place?

Gradle automatically recognises a directory named `buildSrc`. It compiles the project inside it and makes the resulting classes and plugins available to the rest of the build.

This makes `buildSrc` very convenient:

- it requires little setup;
- its plugins are available automatically;
- it keeps shared logic out of module build scripts;
- and it can contain tests like any other Gradle project.

For a small or medium-sized build, this might be all you need. However, `buildSrc` is also *[special](https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html#sec:using_buildsrc)*. Only one can exist at the root of a build, and its output is added to the classpath of every project build script. **Changes to its code can invalidate the configuration phase and make a large part of the build out of date.**

## How do we use an included build?

We can turn the `buildSrc` into a regular, explicitly included Gradle build project, by renaming it to something like `build-logic` and including it in the root `settings.gradle.kts` `pluginManagement {}` block using `includeBuild("build-logic")`. Consuming the precompiled plugin doesn't need to change! This separation gives the build logic an explicit boundary. It is now an independent build with its own dependencies, lifecycle, and tests.

Composite builds are useful for more than build logic, but sharing plugins is one of their primary use cases. Read more about [composite builds](https://docs.gradle.org/current/userguide/composite_builds.html).

See this in [[Step 2] Move from buildSrc to build-logic /w precompiled scripts](https://github.com/anthonymonori/sample-gradle-project/pull/4).

If you have a small project and want the simplest possible setup, `buildSrc` remains a reasonable choice. If the build logic is growing, has dedicated ownership, or needs a clearer classpath and project boundary, an included build is likely a better fit.

## What is the difference between precompiled scripts and binary plugins?

Our `sample.android.library.gradle.kts` is already a plugin. Gradle generates a plugin class from the script and applies it using the filename-derived ID (you can see this generated code yourself). As the logic becomes more complex, we might prefer defining the plugin class ourselves.

Defining a [binary plugin](https://docs.gradle.org/current/userguide/implementing_gradle_plugins_binary.html#header) gives the implementation a clearer structure. We can split responsibilities across regular classes, use explicit types, isolate optional features, and test individual behaviours more easily — just like with our product source code. It also gives us greater control over how the plugin reacts to other plugins and which parts of its implementation become public.

See this in [[Step 3] Replace precompiled scripts with binary convention plugins](https://github.com/anthonymonori/sample-gradle-project/pull/3).

This does not change the experience of the consuming module. It still applies the same convention plugin by ID. We are only changing how that convention is implemented behind the boundary.

## Does build logic only exist in project build scripts?

So far, we have focused on module configuration. However, a large amount of important build logic also lives in `settings.gradle.kts`. Settings may define repository policy, plugin resolution, build-cache configuration, feature previews, project discovery, and included builds. If this logic needs to be shared across multiple repositories, we can place it behind a settings plugin in exactly the same spirit as our project conventions. 

See this in [[Step 4] Extract settings logic into a binary convention plugin](https://github.com/anthonymonori/sample-gradle-project/pull/2).

## What happens when modules need different capabilities?

Our Android library convention works well while every module requires the same configuration.
But not every Android module is identical. Some use Compose. Some use Dagger or Metro. Others might require kotlinx.serialization, SQLDelight, publishing, screenshot testing, or additional testing infrastructure.

We could ask each module to apply and configure the underlying plugins directly, but doing so leaks the implementation of those capabilities into every module. If we later migrate from KAPT to KSP, replace a plugin, update the Compose configuration or change which dependencies make up our convention, every consuming module needs to understand that migration.

This is a custom Gradle DSL:

```kotlin
plugins {
    id("sample.android.library")
}

sample {
    features {
        compose()
    }
}
```

The module no longer needs to know which plugins implement Compose support, or which dependencies our project considers standard. **It declares the intent, which the build logic turns into configuration.**

## Why provide a custom DSL?

A focused custom DSL can make build configuration:

- declarative;
- discoverable through IDE completion;
- consistent across modules;
- and easier to migrate centrally.

For example, we could expose the compiler choice without asking modules to apply KAPT or KSP themselves:

```kotlin
sample {
    features {
        dagger {
            compiler = Compiler.KSP
        }
    }
}
```

The build logic can then apply the correct plugin and dependency configuration. Later, if we later make KSP the default, we can change that convention in one place. Modules that do not require a special override remain untouched. 

See this in [[Step 5] Introduce a custom DSL for opt-in build features](https://github.com/anthonymonori/sample-gradle-project/pull/1).

*In the next post, I’ll go into more detail about how to define a custom Gradle DSL, model its configuration lazily, and turn those declarations into plugin behaviour.*

## Where should your build logic live?

As is often the case with software engineering questions, the answer is: *it depends*.

For a small project, duplicated configuration might still be the most understandable solution.

As repetition grows, a precompiled script plugin gives us a lightweight way to introduce conventions. buildSrc provides a convenient home for that logic, while an included build creates a clearer and more scalable boundary.

As the conventions become more complex, binary plugins let us organise the implementation as regular Kotlin code.

Finally, when modules need to select higher-level capabilities, a small custom DSL can become the public interface to that build logic. You do not necessarily need to reach the final step. Each abstraction should earn its place by removing a real source of repetition, inconsistency, or migration cost.

You can explore the complete progression in the [sample Gradle project](https://github.com/anthonymonori/sample-gradle-project), with each step preserved as a pull request.

---

Thanks for reading along. I can assure you this blog post was written by me, and not by an AI :).
