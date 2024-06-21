---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Nixing your system
info: |
  ## A guide to fun and profit
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# Nixing your system

A guide to fun and profit

<div class="pt-12">
  <!-- <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span> -->
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
---

# What Is Nix?

Nix is a powerful package manager and build system that provides reproducible and declarative environments for software development and deployment.

## Key Features

- **Reproducibility**: Ensures that builds are identical regardless of the environment.
- **Isolation**: Each package is stored in its own unique directory, preventing dependency conflicts.
- **Declarative Configuration**: System configurations are described in a high-level, declarative manner.
- **Rollbacks**: Easy to roll back to previous versions of packages or system configurations.
- **Multi-user Support**: Allows multiple users to share the same system without interference.

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Emphasize reproducibility and why it's useful in software build systems and supply chain management.
-->

---
transition: slide-up
title: What Is Nix?
level: 2
---

## How It Works

- **Functional Approach**: Nix uses a purely functional approach to package management, meaning that the result of building a package depends only on its inputs.
- **Store**: Packages are stored in the Nix store, typically located at `/nix/store`, with unique hashes.
- **Derivations**: Packages are built from _derivations_, which are descriptions of how to build a package.
- **Expressions**: Packages and configurations are described using Nix expressions, written in the Nix Expression Language.

## Use Cases

- **Development Environments**: Create isolated and reproducible development environments.
- **Continuous Integration**: Ensure consistent builds across different CI environments.
- **System Configuration**: Manage entire system configurations declaratively with NixOS.

<!--
Explain a bit more about functional programming and how it's applied in Nix.
Functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data.
How it helps in package management and system configuration.
 -->

---
transition: fade-out
---

# nix-shell and the `/nix/store`

If you want to try out a package, you just can!

## Features

- **Temporary Environment**: Creates a temporary shell environment with the specified packages.
- **Isolation**: Ensures that the environment is isolated from the host system. We can try it out!
- **Immutable**: The nix store is immutable
- **No conflicts**: A package's path is determined by its hash, so there are no conflicts. Either the paths are different due to different `inputs`, or the package is deduplicated.
- **Garbage Collection**: The nix store is garbage collected, so you don't have to worry about disk space.

<!--
For example, cowsay isn't installed in my VM right now:
```
nix-shell -p cowsay
```

How does that work? From that shell, let's find out where cowsay is in $PATH:
```
which cowsay
```

Let's find out exactly what cowsay refers to when running. It's not a dynamic executable, so I'll use strace to find out what it's doing:
```
strace cowsay "Moo" 2>&1 | grep --fixed-strings '"/'

```

It refers almost exclusively to things under `/nix/store`! In fact, the only things it refers to that aren't under `/nix/store` are:
```
access("/etc/ld-nix.so.preload", R_OK)  = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/dev/urandom", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/dev/urandom", O_RDONLY|O_CLOEXEC) = 3
```

`/nix/store` is immutable, by the way:
```
touch /nix/store/lqz6hmd86viw83f9qll2ip87jhb7p1ah-glibc-2.35-224/lib/libdl.so.2
```

Also, what's that long hash about it? It's derived from the inputs, which means, paths either never clash (because they have different inputs), or they deduplicate.

But also, you can have several versions of something and they won't conflict. In my little nix-shell, I already have two versions of glibc, for example

```
find /nix/store/*glibc*/ -name 'libdl.so'
```

This is all temporary though: if we exit out of our nix-shell with Ctrl-D, we can't execute cowsay anymore. Or can we?

```
/nix/store/azn0g0m6yg6m9vmdp3wq6wjbsd1znv44-cowsay-3.7.0/bin/cowsay "Look who's still here"
```

It does, until you garbage collect!

```
nix-collect-garbage
```
 -->

---
transition: fade-out
---

# Installing packages with `nix profile`

Similar to a traditional package manager, `nix profile` allows you to install packages globally.
However, this is not the recommended way to use Nix, as it does not provide the same benefits as a declarative setup.

## Caveats

- **Non-Reproducibility**: Difficult to ensure the same environment across different systems or over time.
- **Complexity**: Managing state changes manually is error-prone and can lead to inconsistencies.
- **Lack of Rollback**: Harder to revert to previous states without a clear record of changes.
- **System-Wide Consistency**: Profiles are user-specific and may lead to inconsistencies across different user environments.

<!--
Explain the difference between using `nix profile` and a declarative setup.

For example, installing cowsay with `nix profile`:
```
nix profile install 'nixpkgs#cowsay'
```

Now this will survive garbage collection, but it's not recommended. Why? Because it's not declarative.
-->

---
level: 2
---

# The Nix Expression Language

Nix has `sets`, which are like dictionaries in Python or objects in JavaScript.

```nix
{
  list =
    [ 1 2 ] ++ [ 3 4 ];
  a = 1;
  b = 2;
}
```

And it has functions, which are called lambdas in some contexts:

```nix
x: x + 1
```
Functions are first-class citizens in Nix, so you can assign them to variables, pass them as arguments, and return them from other functions. (Note that one function can only take one argument but currying is possible.)
```nix
{
    add = a: b: a + b;
    sub = a: b: a - b;
}
```

<!-- 

 -->

---
transition: fade-out
---

# More about sets

````md magic-move {lines: true}
```nix
# Sets can be recursive, so you can have a set that refers to itself:
{
    a = 1;
    b = 2;
}
```

```nix {all|2}
# Note the rec builtin here, which is required for recursive sets
rec {
    a = 1;
    b = 2;
    c = a + b;
}
```

```nix
# We can also use let bindings to make variables available in the `in` block
let
    a = 1;
    b = 2;
in
{
    c = a + b;
}
```

```nix
# Sets can be nested, so you can have a set of sets:
{
    inner = {
        a = 1;
        b = 2;
    };
}
```

```nix
# You can access properties of a set using the `.` operator:
{
    inner = {
        a = 1;
        b = 2;
    };
    result = inner.a + inner.b;
}
```

```nix
# Shorthand for defining a nested set with properties:
{
    inner.a = 1;
    inner.b = 2;
    result = inner.a + inner.b;
}
```


````

<v-click>

## Summary

- `()` are just like in C/Rust/whatever: they mean "evaluate that first"
- `{}` defines a set/object/dictionary/key-value map. it has key = value; inside (each item ends with a semicolon)
- `[]` defines a list/array: it has `1 2 3` inside (space-separated)
- `foo.bar` accesses property `bar` of set `foo`
- `let` lets you bind some names and use them in the in block later, which is preferred to using a recursive set.

</v-click>