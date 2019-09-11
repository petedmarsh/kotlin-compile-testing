<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/74/Kotlin-logo.svg/512px-Kotlin-logo.svg.png" align="right"
     title="Kotlin Logo" width="120">

# Kotlin Compile Testing

[![](https://jitpack.io/v/tschuchortdev/kotlin-compile-testing.svg)](https://jitpack.io/#tschuchortdev/kotlin-compile-testing)
![GitHub](https://img.shields.io/github/license/tschuchortdev/kotlin-compile-testing.svg?color=green&style=popout)
![Maintenance](https://img.shields.io/maintenance/yes/2019.svg?style=popout)
[![Generic badge](https://img.shields.io/badge/contributions-welcome-green.svg)](https://shields.io/)
[![Build Status](https://travis-ci.com/tschuchortdev/kotlin-compile-testing.svg?branch=master)](https://travis-ci.com/tschuchortdev/kotlin-compile-testing)
[![Build status](https://ci.appveyor.com/api/projects/status/jj639rc6whehaf9o?svg=true)](https://ci.appveyor.com/project/tschuchortdev/kotlin-compile-testing)

A library for in-process compilation of Kotlin and Java code, in the spirit of [Google Compile Testing](https://github.com/google/compile-testing). For example, you can use this library to test your annotation processors. 

## Use Cases

- Compile Kotlin and Java code in tests
- Test your annotation processors
- Test your compiler plugins
- Test Kotlin code generation

## Example

Create sources

```Kotlin
class TestEnvClass {}

@Test
fun `test my annotation processor`() {
    val kotlinSource = SourceFile.new("KClass.kt", """
        class KClass {
            fun foo() {
                // Classes from the test environment are visible to the compiled sources
                val testEnvClass = TestEnvClass() 
            }
        }
    """)   
      
    val javaSource = SourceFile.new("JClass.java", """
        public class JClass {
            public void bar() {
                // compiled Kotlin classes are visible to Java sources
                KClass kClass = new KClass(); 
            }
    """)
```
Configure compilation
```Kotlin
    val result = KotlinCompilation().apply {
        sources = listOf(kotlinSource, javaSource)
        
        // pass your own instance of an annotation processor
        annotationProcessors = listOf(MyAnnotationProcessor()) 
        
        inheritClassPath = true
        messageOutputStream = System.out // see diagnostics in real time
    }.compile()
```
Assert results
```Kotlin
    assertThat(result.exitCode).isEqualTo(ExitCode.OK)	
    
    // Test diagnostic output of compiler
    assertThat(result.messages).contains("My annotation processor was called") 
    
    // Load compiled classes and inspect generated code through reflection
    val kClazz = result.classLoader.loadClass("KClass")
    assertThat(kClazz).hasDeclaredMethods("foo")
}
```


## Features
- Mixed-source sets: Compile Kotlin and Java source files in a single run
- Annotation processing: 
    - Run annotation processors on Kotlin and Java sources
    - Generate Kotlin and Java sources
    - Both Kotlin and Java sources have access to the generated sources
    - Provide your own instances of annotation processors directly to the compiler instead of letting the compiler create them with a service locator
    - Debug annotation processors: Since the compilation runs in the same process as your application, you can easily debug it instead of having to attach your IDE's debugger manually to the compilation process
- Inherit classpath: Compiled sources have access to classes in your application
- Project Jigsaw compatible: Kotlin-Compile-Testing works with JDK 8 as well as JDK 9 and later
- JDK-crosscompilation: Provide your own JDK to compile the code against, instead of using the host application's JDK. This allows you to easily test your code on all JDK versions
- Find dependencies automatically on the host classpath

## Installation <img src="https://i.imgur.com/iV36acM.png" width="23">

Add jitpack to the repositories in your root `build.gradle` file:

```Groovy
allprojects {
	repositories {
		// ...
		maven { url 'https://jitpack.io' } // add this
	}
}
```

Add dependency to your module `build.gradle` file:

```Groovy
dependencies {
        // ...
	implementation 'com.github.tschuchortdev:kotlin-compile-testing:1.2.0'
}
```

<img src="https://emojipedia-us.s3.dualstack.us-west-1.amazonaws.com/thumbs/120/whatsapp/186/white-medium-star_2b50.png" width="23"> Remember to leave a star if you found it useful <img src="https://emojipedia-us.s3.dualstack.us-west-1.amazonaws.com/thumbs/120/whatsapp/186/white-medium-star_2b50.png" width="23">

## License

Copyright (C) 2019 Thilo Schuchort

Licensed under the Mozilla Public License 2.0

For custom license agreements contact me directly 
