+++
title = "Introducing Myco: a secure JS runtime"
date = 2023-05-07T00:22:25-06:00

[taxonomies]
tags = ["Languages", "Effects", "Capabilities"]
+++

A few months back I wrote about [Log4Shell and its implications for language design](22_12_06_effects_capabilities_and_log4shell). Since then I've worked on a number of language concepts based around these ideas, but always hit a wall. There is *so much* to build when you're starting a language from scratch, and I never made it to the effects or capabilities concepts that I wanted to explore.

Yesterday I stumbled upon [a blog post by Deno](https://deno.com/blog/roll-your-own-javascript-runtime) where they demonstrate how to use the `deno_core` crate to implement new JS runtimes on top of Deno. After following along with the blog post I realized that they provided all the tools needed to start an experiment in object-capabilities, so over the past 24 hours I built out enough of a proof of concept that I feel ready to demo it and explain my goals for the future.

<img style="height: 6em;" src="/img/23-05/myco-logo.png" alt="Myco logo"/> 

## Myco: Object-Capabilities for JavaScript

[Myco](https://github.com/mycojs/myco) is a new, experimental JavaScript runtime designed to provide a clearly visible flow of authority. In Myco, all functions capable of causing external effects (such as network or file access) are enclosed in the `Myco` object, which is not made available globally. Instead it's passed in to the default export of the main module, like this:

```typescript
// index.ts
export default async function ({files, console}: Myco) {
    const path = "./log.txt";
    try {
        const readToken = await files.requestRead(path);
        const contents = await readToken.read();
        console.log("Read from a file", contents);
    } catch (err) {
        console.error("Unable to read file", path, err);
    }
}
```

Since there's no way to import the `Myco` object it has to be passed down from this main function to child functions.

### Scoped access: passing tokens

This feature is what sets Myco apart from other runtimes like Node and Deno: security is considered per-scope, not per-application. In Node or Deno, the entire application has access to do something or it doesn't. If you need access to a privileged API in your code, you have to give it to all of your libraries as well. This is fine if you trust them, but we've repeatedly seen that [libraries cannot always be trusted](https://dl.acm.org/doi/pdf/10.1145/358198.358210).

In Myco, you can delegate a thin slice of your application's permissions to a library. For example, if we have a library that needs to load its data from a file, we *could* let it access the whole filesystem, but that would be overkill and could lead to security vulnerabilities. Instead, we can grant the library read access to the one file that it cares about:

```typescript
// index.ts
import {readDataFromFile} from "./somewhat-trustworthy-lib"

export default async function ({files}: Myco) {
    const path = "./stored-data.json";
    const readToken = await files.requestRead(path);
    const libraryObj = await readDataFromFile(readToken);
    // Do something with the libraryObj
}
```

Inside the library, the function:

```typescript
// somewhat-trustworthy-lib.ts
export async function readDataFromFile(
    readToken: Myco.Files.ReadToken
) {
    const fileContents = await readToken.read();
    const data = parse(fileContents);
    return data;
}
```

Because access to the filesystem is restricted to those functions that have access to the `Myco.Files` object, our library's permissions are limited to reading the specific file that we passed a token for: `./stored-data.json`. It cannot read from any other files, and it cannot write to *any* files.

It would be a bit of a pain to have to pass in every single file that a library needs access to, so Myco provides a way to give a handle on a whole directory:

```typescript
import {LibClass} from "./somewhat-trustworthy-lib"

export default async function ({files}: Myco) {
    const directory = await files.requestReadWriteDir("./data");
    const lib = new LibClass(directory);
    // Do stuff with lib
}
```

Inside of `LibClass`, the library can use the directory token to read and write any files within the directory, but not outside of it.

### Unscoped access: passing the Myco object

If we want to grant permission to perform arbitrary file I/O to a child function, we can just give it the whole `files` object. This is best suited for internal code, and would not be appropriate in a library API.

Here's how that would work:

```typescript
// index.ts

import {childFunction} from "./module"

export default async function ({files, console: _console}: Myco) {
    console = _console;
    await childFunction(files);
}

// module.ts

export async function childFunction(files: Myco.Files) {
    const path = "./log.txt";
    try {
        const readToken = await files.requestRead(path);
        const contents = await readToken.read();
        console.log("Read from a file", contents);
    } catch (err) {
        console.error("Unable to read file", path, err);
    }
}
```

The trust level is higher in a call like this, because arbitrary files can be accessed, but it's also more flexible.

## What's next for Myco

The current iteration of Myco is *very* much a prototype, but I'm very excited about where it's going. I'm still pretty new to Rust, so I expect that the code could use a lot of improvements.

Production-time compatibility with the existing NPM ecosystem is a non-goal for Myco because the scoped security model leads to a dramatically different philosophical approach to dependencies. [The tall dependency towers](https://xkcd.com/2347/) that we've come to expect on NPM are impractical when permissions have to be explicitly passed down the tree, so I expect to see broader, shallower dependency graphs that require few permissions.

One of my first goals is to figure out what we *will* be using to manage packages and as replacement build tooling. Getting TypeScript working is a first priority.

I'm thinking to start working through these questions by getting a proper web server running in Myco. Aside from helping me to work through the tooling questions, running a server of moderate complexity should help me to recognize the weak parts of the model and develop solutions to the ergonomic problems, as well as giving us something to benchmark performance-wise.

I'm pretty happy with what I have so far and excited to move forward with the tooling!
