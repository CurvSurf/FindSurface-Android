# FindSurface for Android

**Curv*Surf* FindSurface™**

## Overview

This library is the implementation of the **FindSurface** library for **Android (Java/Kotlin).**



## Samples

These are the samples to help you get started to make your application with the library (more samples are to be added in the future):

- [BasicDemo (Kotlin)](https://github.com/CurvSurf/FindSurface-BasicDemo-Android)
- BasicDemo (NDK C/C++) - to be updated in the future.



## F.A.Qs

### Q. How do I import this library to my project?

Download and unzip [the library release](https://github.com/CurvSurf/FindSurface-Android/releases/) and add the following depedency in your module's `gradle.build` file:

````kotlin
dependencies {
  	implementation files('<the-downloaded-aar-file-path>/findsurface.aar')
}
````

A recommended path is `libs/findsurface.aar` to make the project self-contained.  

And then, click the "Sync Project with Gradle Files" button on the top left corner of the Android Studio.

That's it. You're all set to go.



### Q. Does this library work on Android Native/NDK?

You have to directly use the header files and the .so file embedded in the provided aar file in order to link the native side. Refer to [this document](https://github.com/CurvSurf/FindSurface-Android/blob/master/How-to-import-the-library-in-NDK-projects.md) for details.

## ---

(c) Copyright 2021 CurvSurf, Inc. All rights reserved.

This library's ownership is solely on CurvSurf, Inc. and anyone can use it for non-commercial purposes. Contact to support@curvsurf.com for commercial use of the library.

