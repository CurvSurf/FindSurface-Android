# How to import the library in NDK projects

> Note: This document describes how to import the FindSurface library in your Android NDK projects. Follow the instructions below only if your projects invoke FindSurface API on the native side (C/C++).

When using FindSurface APIs in NDK projects, `libFindSurface.so` file that contains the native APIs is required to link the APIs invoked in the native side to the projects. Even though the file is already embedded in [`findsurface.aar`](), it is still needed for the linking process. As the file is required only for the build-time, you can access the file by extracting it from the aar file, as an intermediate input for the linking process, as in [the Google ARCore library](). 



## Getting the Header Files

You have to download the header files [here]() because the headers for the native side (C/C++) are not included in the .aar file. Put the files in your source directory or other places (additional steps required).



## Extracting the Native Library

You could get the file by manually unzipping the aar file after converting the file extension into '.zip'. Instead, there is a better way. [This Google ARCore document]() introduces a task that automatically extracts the temporary so file for the build process. If you are already using ARCore, you will find that ARCore does the same things for itself. In that case, take advantage of the things done by ARCore.



### Path Variables

First of all, declare the following variables in your module gradle (`app/build.gradle`):

````kotlin
def findsurface_libpath = "<path-to-the-aar>/findsurface.aar"
def native_libpath = "${buildDir}/native-libs"
def findsurface_include = "<path-to-the-headers>"
````

`native_libpath` is a path for CMake to search for so files to link its dependent libraries.

Specify [the path where you put the header files](#getting-the-header-files) to `findsurface_include`.

All path variables are just for your convenience. You can omit them and specify the paths to everywhere they are needed instead (not recommended).



If you're using ARCore, you will see the following statement:

````Kotlin
def arcore_libpath = "${buildDir}/arcore-native"
````

In that case, remove the statement since `native_libpath` will work for both libraries. Otherwise, you have to declare an additional copy command in the task that we will discuss later.



### Configuration and Dependencies

Create a gradle configuration that marks which aars to extract .so files from:

````kotlin
configurations { natives }
````

Here, `natives` is used as a mark indicating which aar files have their native libraries in them.



Add the dependencies for both the Java and the native libraries:

````kotlin
dependencies {
    implementation files( findsurface_libpath ) // for FindSurface APIs used in Java/Kotlin
    natives files( findsurface_libpath )  // indicates the aar has native libraries

    ...
}
````



You may find these dependencies too if ARCore:

````kotlin
dependencies {
    ...

    implementation 'com.google.ar:core:1.24.0'
    natives 'com.google.ar:core:1.24.0'
    
    ...
}    
````

These dependencies with `native` configuration allow tasks to access a list of the aar files that contain native libraries.



### Tasks

Create a task that copies the native libraries from the aar file:

````kotlin
task extractNativeLibraries() {
    outputs.upToDateWhen { false }
    
    doFirst {
        configurations.natives.files.each { f ->
            copy {
                from zipTree(f)
                into native_libpath
                include "jni/**/*"
            }
        }
    }
}

````

`configurations.natives.files` is the list of the aar files that contain native libraries so the task unzips (`zipTree`) and copies the contents into `native_libpath` for each of the aar files.

The task may already exist if you're using ARCore. In that case, you just have to change `into arcore_libpath` to `into native_libpath`.



Add the following statement to invoke the task only for building the native side:

````kotlin
tasks.whenTaskAdded {
    task-> if (task.name.contains("external") && !task.name.contains("Clean")) {
        task.dependsOn(extractNativeLibraries)
	}
}
````



### CMake

The CMake, which is responsible to build the native side, must be informed from the gradle, about the paths to the native libraries:

````kotlin
android {
    ...
    externalNativeBuild {
        cmake {
            cppFlags "-std=c++14", "-Wall"
            arguments "-DANDROID_STL=c++_static",
            		  "-DNATIVE_LIBPATH=${native_link_libpath}/jni",
            		  "-DFINDSURFACE_INCLUDE=${findsurface_include}"
        }
    }
    ...
}
````

The arguments are macro definitions informing CMake paths to locations. In case of using ARCore, append the arguments above to the next of the existing arguments:

````kotlin
android {
    ...
    externalNativeBuild {
        cmake {
            cppFlags "-std=c++14", "-Wall"
            arguments "-DANDROID_STL=c++_static",
                      "-DNATIVE_LIBPATH=${native_libpath}/jni",
            		  "-DARCORE_INCLUDE=<path-to-the-arcore-headers>",
            		  "-DFINDSURFACE_INCLUDE=${findsurface_include}"
        }
    }
    ...
}
````

You may remove the `-DARCORE_LIBPATH` or set the same path to it. There will be the ARCore native libraries in the `NATIVE_LIBPATH` as [we modified the path in the task](#tasks).



Finally, in CMakeLists.txt, add the following statements:

````cmake
add_library(
	FindSurface # declare 
	SHARED
	IMPORTED
)
set_target_properties(
	FindSurface // refer to the declaration in add_library
	PROPERTIES
	IMPORTED_LOCATION ${NATIVE_LIBPATH}/${ANDROID_ABI}/libFindSurface.so
	INTERFACE_INCLUDE_DIRECTORIES ${FINDSURFACE_INCLUDE}
)

...

target_link_libraries( # Specifies the target library.
					   native-lib # should be your native library name
					   
					   # libFindSurface
					   FindSurface # refer to the declaration in add_library
					   
					   ... # other libraries
					   
					   # libarcore if used
					   arcore
)
````

In the case of using ARCore, `ARCORE_LIBPATH` should be changed to `NATIVE_LIBPATH` if you removed the `-DARCORE_LIBPATH` declaration in the gradle.

Refer to [the Google document](https://developer.android.com/ndk/guides/cmake) for more details about CMake.



## Putting All Together

**Gradle (module)**

````kotlin
def findsurface_libpath = "<path-to-the-aar>/findsurface.aar"
def native_link_libpath = "${buildDir}/native-link"
def findsurface_include = "<path-to-the-headers>"

...

// creates a configuration to mark which aar files to extract .so files from
configurations { natives }

android {
    ...
    externalNativeBuild {
        cmake {
            cppFlags '-std=c++14'
            arguments "-DANDROID_STL=c++_static",
            		  "-DNATIVE_LIBPATH=${native_link_libpath}/jni",
            		  "-DFINDSURFACE_INCLUDE=${findsurface_include}"
        }
    }
    ...
}


dependencies {
    implementation files( findsurface_libpath )
    natives files( findsurface_libpath )

    ...
}

task extractNativeLibraries() {
    outputs.upToDateWhen { false }
    doFirst { // do before the build
        configurations.natives.files.each { f ->
            copy {
                from zipTree(f)
                into native_link_libpath
                include "jni/**/*"
            }
        }
    }
}

tasks.whenTaskAdded {
    task-> if (task.name.contains("external") && !task.name.contains("Clean")) {
        task.dependsOn(extractNativeLibraries)
    }
}
````

**CMakeLists.txt**

````cmake
add_library(
	FindSurface
	SHARED
	IMPORTED
)
set_target_properties(
	FindSurface
	PROPERTIES
	IMPORTED_LOCATION ${NATIVE_LIBPATH}/${ANDROID_ABI}/libFindSurface.so
	INTERFACE_INCLUDE_DIRECTORIES ${FINDSURFACE_INCLUDE}
)

...

target_link_libraries( # Specifies the target library.
					   native-lib
					  
					   # libFindSurface
					   FindSurface
					   
					   ...
)
````



## Putting All Together (with ARCore)

**Gradle (module)**

````kotlin
def findsurface_libpath = "<path-to-the-aar>/findsurface.aar"
def native_link_libpath = "${buildDir}/native-link"
def findsurface_include = "<path-to-the-headers>"

...

// creates a configuration to mark which aar files to extract .so files from
configurations { natives }

android {
    ...
    externalNativeBuild {
        cmake {
            cppFlags ""-std=c++14", "-Wall"
            arguments "-DANDROID_STL=c++_static",
                      "-DNATIVE_LIBPATH=${native_libpath}/jni",
                      "-DARCORE_INCLUDE=${project.rootDir}/../../libraries/include",
            		  "-DFINDSURFACE_INCLUDE=${findsurface_include}"
        }
    }
    ...
}


dependencies {
    implementation files( findsurface_libpath )
    natives files( findsurface_libpath )

    implementation 'com.google.ar:core:1.24.0'
    natives 'com.google.ar:core:1.24.0'
    
    ...
}

task extractNativeLibraries() {
    outputs.upToDateWhen { false }
    doFirst { // do before the build
        configurations.natives.files.each { f ->
            copy {
                from zipTree(f)
                into native_link_libpath
                include "jni/**/*"
            }
        }
    }
}

tasks.whenTaskAdded {
    task-> if (task.name.contains("external") && !task.name.contains("Clean")) {
        task.dependsOn(extractNativeLibraries)
    }
}
````

**CMakeLists.txt**

````cmake
add_library( FindSurface SHARED IMPORTED )
set_target_properties(
	FindSurface
	PROPERTIES
	IMPORTED_LOCATION ${NATIVE_LIBPATH}/${ANDROID_ABI}/libFindSurface.so
	INTERFACE_INCLUDE_DIRECTORIES ${FINDSURFACE_INCLUDE}
)

add_library( arcore SHARED IMPORTED )
set_target_properties(
	arcore
	PROPERTIES
	IMPORTED_LOCATION ${NATIVE_LIBPATH}/${ANDROID_ABI}/libarcore_sdk_c.so
	INTERFACE_INCLUDE_DIRECTORIES ${ARCORE_INCLUDE}
)
...

target_link_libraries( # Specifies the target library.
					   native-lib
					  
					   # libFindSurface
					   FindSurface
					   
					   ...
					   
					   # libarcore
					   arcore
)
````

