---
layout: post
title: 'AGP tidbits: Details product variant'
date: 2024-05-06 15:59
permalink: /agp-tidbits-default-product-variant/
tag: [blog, android]
categories: [android]
author: Antal Monori
description: A quick guide on how to set a default Active Build Variant in Android Studio 
---

![image](assets/posts/2024-05-06-agp-tidbits-default-product-variant/header.png)

How often do you consider the experience of opening up your project in Android Studio for the first time? If you are in a small team, likely not too often. If you look after the developer experience of a large Android engineering team, you probably do!


Build variants can help you to define different versions of your app, like targeting different environments (production vs staging), audience (free vs paid), or different device variations. One common use case I see often, is limiting the resource you bundle to a given language and screen density for faster compilation times. Read more [here](https://developer.android.com/studio/build/build-variants).


The above product flavor variations can create quite a long and complex list of Build Variants to choose from in Android Studio:

![image](assets/posts/2024-05-06-agp-tidbits-default-product-variant/build-variants-before.png)

As you can notice, **this list is simply sorted in ascending alphabetical order** — and therefore the first one might not be the best configuration for someone just starting up with your project. If you want to encourage the use of `staging` over `production`, or `external` over `dev` by default, then please read on.

## How do I assign a different default?


Thankfully, Android Gradle Plugin provides a DSL just for this since 7.1.0, part of their `gradle-api` artifact:

```kotlin
interface ApplicationProductFlavor : ApplicationBaseFlavor, ProductFlavor {
    /** Whether this product flavor should be selected in Studio by default  */
    var isDefault: Boolean
}
```


Source: [https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:build-system/gradle-api/src/main/java/com/android/build/api/dsl/ApplicationProductFlavor.kt;l=54?q=isDefault](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:build-system/gradle-api/src/main/java/com/android/build/api/dsl/ApplicationProductFlavor.kt;l=54?q=isDefault)

All you have to do, is to assign `isDefault = true` to each dimension to tell Studio the order of your preference as it creates the final Build Variant. No new dependencies needed!

Let's take an example configuration below:

```kotlin
android {
    flavorDimensions.addAll(listOf("audience", "api"))

    productFlavors {
        create("production") {
            dimension = "api"
            buildConfigField("string", "ROOT_URL", "https://acme-v02.api.letsencrypt.org")
        }

        create("staging") {
            dimension = "api"
            isDefault = true
            buildConfigField("string", "ROOT_URL", "https://acme-staging-v02.api.letsencrypt.org")
        }

        create("dev") {
            isDefault = true
            dimension = "audience"
            applicationIdSuffix = ".internal"
            buildConfigField("boolean", "EXTERNAL", "false")
            resourceConfigurations.addAll(listOf("en", "xxhdpi"))
        }

        create("external") {
            dimension = "audience"
            buildConfigField("boolean", "EXTERNAL", "true")
        }
    }
}
```


The above scenario will ensure that out of the api dimension staging will be preferred over production, and out of the audience dimension dev will be preferred over external.

Next time when an engineer clones your repository, or invalidates the module files for the application module, it would result in having devStagingDebug pre-selected to avoid any confusion.

![image](assets/posts/2024-05-06-agp-tidbits-default-product-variant/build-variants-after.png)

### How do I test this?

If you inspect you application's generated module file (e.g. `.idea/modules/app/project-name.app.iml`), you'll find that this is where AGP injects the relevant information to tell Studio which one to prefer in the UI.

![image](assets/posts/2024-05-06-agp-tidbits-default-product-variant/how-do-i-test.png)

If you clear the right section, you should be ready to let Studio sync once again and you'll see the results.
----

Thanks for reading along and make sure to follow me if you are interested in more findings as I uncover the nits and grits of Android Gradle Plugin in the future.
