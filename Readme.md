# FindSurfaceFramework for Android

**Curv*Surf* FindSurface™**

## Overview

This library is the implementation of the FindSurface library for Android (Java/Kotlin).



## Samples

These are the samples to help you get started to make your application with the framework (more samples are to be added in the future):

- [BasicDemo (Kotlin)](TBD)



## F.A.Qs

### Q. How do I import this framework to my project?

Download [the library](TBD) and add the following depedency in your module's `gradle.build` file:

````kotlin
dependencies {
  	implementation files('<the-downloaded-aar-file-path>/findsurface.aar')
}
````

And then, click the "Sync Project with Gradle Files" button on the top left corner of the Android Studio.

That's it. You're all set to go.



### Q. Does this framework work on Android Native/NDK?

You have to directly use the header files and the .so file embedded in the provided aar file in order to link the native side. If your app has both native part (c/c++) and Java/Kotlin part, use the aar file when you build the app. Details on the steps will be updated in the future.

## ---

(c) Copyright 2021 CurvSurf, Inc. All rights reserved.

This library's ownership is solely on CurvSurf, Inc. and anyone can use it for non-commercial purposes. Contact to support@curvsurf.com for commercial use of the library.

