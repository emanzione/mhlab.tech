---
title: "Call into Rust from C# and Unity"
date: 2019-08-24

categories: ['Game Development']
tags: ['Rust', 'Performance', 'Optimizations']
author: "Emanuele Manzione"
noSummary: false

resizeImages: false
---
Rust is a safe and fast low-level language and recently I got enthusiastic about it. So I would like to add it in my game-dev pipeline. Today I will try to use a Rust-based library in Unity3D.
<!--more-->

#### Getting started

I assume that you already know a little bit how to work with Rust and Cargo, so I will not speak about how to install the compiler and its environment.

I create a new lib project with `cargo init --lib` and specify in Cargo.toml that I want a dynamic library as output:

```toml
[lib]
name = "unity_rust"
crate-type = ["dylib"]
```

To know more about this topic, click [here](https://doc.rust-lang.org/reference/linkage.html).

I also create a new Unity project for the sake of testing.

#### Writing some Rust

Now it's time to write some Rust. I want start simple: just a function to generate a random int between 0 and 100. I am using the `rand` crate, so be sure to add it as dependency.

```csharp
#[no_mangle]
pub extern fn get_random_int() -> i32 {
    let mut rng = rand::thread_rng();
    rng.gen_range(0, 100)
}
```

There are two interesting aspects to point out. The first one is the `#[no_mangle]` attribute: it informs the compiler not to mangle the symbol name for my function, in this way I can easily refer to it by name later.
The second one is the `extern` keyword: it specifies that the function will be exported with the C function call convention.

Now I run `cargo build` or `cargo build --release` and the compiled library will be generated in my `target` folder.

#### On Unity side

I create a `Plugins` folder in the Unity project and put there the compiled native DLL. So I create the C# interop code:

```csharp
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

public static class RustRandom
{
    [DllImport("unity_rust")]
    private static extern int get_random_int();

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public static int GetRandomInt()
    {
        return get_random_int();
    }
}
```

Something to point out here too. I used `DllImport` to inform the compiler about the name of the assembly that contains the following symbol. I used `extern` to inform the compiler that it is an external function. I wrapped the imported function in a public `GetRandomInt` that matches the C# naming convention. I used `MethodImpl` to suggest an aggressive inlining, since the function is just a wrapper.

#### Time to test

Now it is time to test what I coded. I create a simple MonoBehaviour to display the random generated int:

```csharp
using UnityEngine;

public class RustRandomTest : MonoBehaviour
{
    private void Start()
    {
        Debug.Log("Random number: " + RustRandom.GetRandomInt());
    }
}
```

Attach this component to a game object and hit the "Play" button. The console should log a random number like: `Random number: 73`.

#### Advanced usage

Well, what I described until now is really basic and it has probably not so many usages. Let's try to dive deeper!

Instantiating a ThreadRng everytime I want a random number is not efficient. The proper way would be to instantiate it once and retrieve it on demand. Also, if I instantiate something I also have to free it when I don't need it anymore.

```csharp
pub struct RandomGeneratorParameters {
    min: i32,
    max: i32
}

pub struct RandomGenerator {
    rng: rand::prelude::ThreadRng,
    parameters: RandomGeneratorParameters
}

#[no_mangle]
pub extern fn create_random_generator(params: RandomGeneratorParameters) -> *mut RandomGenerator {
    unsafe { transmute(Box::new(RandomGenerator {
        rng: rand::thread_rng(),
        parameters: params
    })) }
}

#[no_mangle]
pub extern fn get_random_int(rng_ptr: *mut RandomGenerator) -> i32 {
    let rng = unsafe { &mut *rng_ptr };
    rng.rng.gen_range(rng.parameters.min, rng.parameters.max)
}

#[no_mangle]
pub extern fn destroy_random_generator(rng_ptr: *mut RandomGenerator) {
    let rng : Box<RandomGenerator> = unsafe { transmute(rng_ptr) };
}
```

The interesting part here is the use of `std::mem::transmute` (you can find additional info about it [here](https://doc.rust-lang.org/std/mem/fn.transmute.html)), that basically works like `memcpy` in C.

`create_random_generator` creates a RandomGenerator instance, box it on the heap, then transmutes it to a `*mut`.

`get_random_int` now accepts a mutable pointer to a RandomGenerator instance.

`destroy_random_generator` accepts the mutable pointer to a RandomGenerator instance, transmutes it back to the boxed RandomGenerator instance and lets Rust deallocate it when it goes out of scope.

Now it is C#'s turn. Its interop code should look like this:

```csharp
using System;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

[StructLayout(LayoutKind.Sequential)]
public struct RustRandomParameters
{
    public int Min;
    public int Max;
}

public static class RustRandom
{
    private static IntPtr RngPtr;
    
    [DllImport("unity_rust")]
    private static extern IntPtr create_random_generator(RustRandomParameters parameters);
    
    [DllImport("unity_rust")]
    private static extern int get_random_int(IntPtr rngPtr);
    
    [DllImport("unity_rust")]
    private static extern void destroy_random_generator(IntPtr rngPtr);
    
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public static void Initialize()
    {
        RngPtr = create_random_generator(new RustRandomParameters()
        {
            Min = 0,
            Max = 100
        });
    }
    
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public static int GetRandomInt()
    {
        return get_random_int(RngPtr);
    }
    
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public static void Dispose()
    {
        destroy_random_generator(RngPtr);
    }
}

```

Nothing relevant, same logic as before. The only thing worth point out is the use of `IntPtr` to represent a native pointer and the `StructLayout(LayoutKind.Sequential)` attribute to make the sequential memory layout of that struct explicit.

And now, again, the test:

```csharp
public class RustRandomTest : MonoBehaviour
{
    private void Awake()
    {
        RustRandom.Initialize();
    }

    private void Start()
    {
        Debug.Log("Random number: " + RustRandom.GetRandomInt());
    }

    private void OnDestroy()
    {
        RustRandom.Dispose();
    }
}
```

It does work! Also, remember that calling into Rust functions has the same cost of calling into C functions from C#.

And that's it. I don't know if it will be useful for anything I will do in the future, but it should be enough to fully interoperate from Unity/C# into Rust.

#### External resources

- https://blog.rust-lang.org/2015/04/24/Rust-Once-Run-Everywhere.html
- https://doc.rust-lang.org/1.30.0/book/2018-edition/ch19-01-unsafe-rust.html
- https://www.reddit.com/r/rust/comments/2fmvcy/rust_ffi_and_opaque_pointer_idiom/
- https://medium.com/jim-fleming/rust-lang-in-unity3d-eeaeb47f3a77
