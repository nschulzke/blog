+++
title = "Effects, Capabilities, and Log4Shell"
date = 2022-12-06T00:22:25-06:00

[taxonomies]
tags = ["Languages", "Effects", "Capabilities"]
+++

One year ago, the Log4J logging library, which is widely used in the Java ecosystem, was hit by the Log4Shell vulnerability. This bug allowed attackers to execute arbitrary code on a server running Log4J by sending a carefully crafted log message.

The vulnerability was caused by a flaw in how Log4J handled log messages that contained JNDI references. JNDI is a Java API that lets applications access naming and directory services, like LDAP and DNS. In the case of Log4Shell, the attacker was able to use JNDI references in a log message to remotely run code on the vulnerable server.

Most Log4J users were shocked to learn that the library could access the internet and execute remote code, since it's usually used for more basic tasks like formatting log messages and rotating log files. The inclusion of JNDI lookups in Log4J was intentional, but few people who were using the library knew about or wanted this feature.

# Trusting trust

Ken Thompson, in his 1983 Turing Award lecture "Reflections on Trusting Trust", warned about trusting code you did not write:

> You can't trust code that you did not totally create yourself. ... No amount of source-level verification or scrutiny will protect you from using untrusted code.[^4]

Thompson argues that a manual audit of even a *small* amount of external code is insufficient to determine if it is trustworthy. If you do not trust the author of the code, you cannot reasonably trust the code, no matter how much you've looked through it.

Today our systems are built with an ever-increasing amount of external, open-source code. This has been a tremendous boon for productivity across the entire industry because we no longer have to reinvent the wheel with each project. What is becoming apparent is that the cost of this speed is that we now trust an increasing number of people with the fate of our own software.

If, as Thompson argues, we cannot rely on manual verification, we have three main options: accept our vulnerability, abandon open source, or find a tool that can reliably verify code safety for us. Unfortunately, our current tools weren't designed to cope with this explosion of third-party code. Most OSes default to assuming that any executable that a given user runs should have the same privileges as any other executable. Our languages, in turn, assume that all code that runs within a single program is equally trustworthy.

These assumptions are no longer valid. Code that is written by an internal team is far more trustworthy than code that comes from further up the supply chain. Even if the OS were to be made more secure, not every privilege granted to our first-party code should be made transitively available to third-party libraries. In order to solve the software supply-chain crisis without losing open source, our languages must adapt to the new environment.

# Language-level security

Trust is the fatal flaw in almost every modern programming language. They assume that *all* code should be allowed to do anything that the OS is willing to allow *any* code to do. In the case of Log4Shell, Java not only grants access to JNDI lookups unless explicitly disabled, it assumes that if *any* code needs access to JNDI then *all* code should have access to it.

This extends to each feature you might use. Do you need to bind to a port? Then all code in your project can bind to a port. Do you need to read from a file? Then all code in your project can read from that file (and if you're not careful with OS permissions, from the entire filesystem). Do you need to talk to external services? Then any library can phone home for any reason.

In order to safely use open-source software, our languages need to fix this flaw. We need to be able to bring in libraries that we trust to a point, but no further. I should be able to add a logging library and be 100% confident that it isn't talking to external servers, much less downloading and executing remote code. I should be able to bring in a linear algebra library and know that it isn't accessing the filesystem. I should be able to look at each call to external code and, from the call site, *know* what it is allowed to do.

There are two language features I am aware of that are perfectly suited to solving this problem: Capabilities and Effects. In the rest of this post I'm going to give a brief introduction to these two concepts, with links for those who wish to explore more. I'll be using some made-up syntax to bolt these concepts onto Java for the sake of example, but I'm not advocating for a particular implementation, just illustrating the concepts.

# Capabilities

Capabilities[^5] are a means of controlling authorization by restricting the use of a sensitive resource to code that has been granted an unforgeable access token. In an Object Oriented language, a reference to an object can (if unforgeable) function as a capability: if you do not have a reference to the object, you cannot call any of its methods. To implement a capabilities-based security model in an OO language, you make it impossible to instantiate security-sensitive objects directly, instead requiring they be passed in from further up the call tree.

In Java today, any piece of code can get a handle for any file for reading or writing, as long as the OS gives permission:

```java
public class ReadFile {
    public static void main(String[] args) {
        File file = new File("top_secret_data.txt");
        // Now I can do anything with this file
    }
}
```

In this example, we're accessing the file directly within the main method, but it would be entirely possible for this exact code to run deep in some library. Compromised code can use this feature to wreak havoc.

In contrast, this is what a capabilities model might look like in Java:

```java
public class ReadFile {
    public static void main(String[] args, Host host) {
        File file = host.requestFile("top_secret_data.txt");
        // Now I can do anything with this file
    }
}
```

In the capabilities version of Java, we remove the public constructor for `File` and replace it with a call to a `Host` object that is passed into `main` by the runtime. The only way to get access to *any* sensitive resource is to request it from the `Host`.

Under this model, Log4J would have had to receive the *capability* to use JNDI from `Host`. This could be done by requesting `Host` directly, but that would put up red flags for security. Instead, they would need to provide a version of `getLogger` that requests a JNDI context:

```java
public class LogExample {
    public static void main(String[] args, Host host) {
        Logger logger = LogManager.getLogger(host.requestJndiContext());
    }
}
```

If the only way to get access to JNDI were to get it through the `Host` object, and the only way to get access to the `Host` object is through the `main` function, then JNDI becomes opt-in and, what's more, finely controlled: no application can accidentally use JNDI, and in any application that *needs* JNDI we can easily verify which modules were granted access.


The major flaw in the capabilities implementation shown above is that there is a lot of boilerplate involved in passing around object handles instead of being able to construct them on the fly deep in nested code. This could result in programmers passing the whole `Host` object far deeper in the code than is strictly necessary, weakening the security of their applications (though it wouldn't ever get weaker than it currently is).

The boilerplate could be aleviated with a feature like Scala 3's context parameters.[^6] The capabilities required would remain in the function signatures, easy to inspect at every level of the code, but wouldn't need to be explicitly provided as long as they're available in the caller.

# Effects

Where capabilities lend themselves well to an object-oriented style, effects originate in the functional programming world. In their original context, they're intended to be used for handling all side effects, but they could also easily be scoped to only handle security-sensitive effects.

An effect system is a way of tracking and managing side effects in a programming language's type system. They are similar to Java's checked exceptions, but more general and flexible. In an effect system, the side effects that a function can perform are specified in its signature. Functions that call other functions can either handle the effects themselves or pass them along to other functions higher up in the call chain. After an effect has been handled, the code that originally performed the effect can resume execution.

You can think of an effect system as a kind of "resumable exception" mechanism. It allows developers to specify and enforce constraints on the ways that code can interact with external resources, such as the file system or the network. Because this information is included in the function signature, it becomes glaringly obvious what sorts of vulnerabilities any given library is at risk of.

For example, loading a file can be implemented as an effect that a function `performs` (which is used the same way as `throws` is for exceptions):

```java-effects
public class ReadFile {
    public static void main(String[] args) performs OpenFile {
        Optional<File> file = perform new OpenFile("top_secret_data.txt");
        // Now I can do anything with this file
    }
}
```

The `perform` keyword is the equivalent of `throw` for exceptions: it hands control up the stack to the next handler, in this case directly to the runtime. The difference is that `perform` is an expression: it evaluates to the result generated by the handler.

The type that it evaluates to is defined as part of the effect:

```java-effects
public class OpenFile implements Effect<Optional<File>> {
    public OpenFile(String pathname) {
        // ...
    }
    // ...
}
```

If Java had an effect system, Log4J could have implemented JNDI lookup as an effect:

```java-effects
public interface JndiLogger {
    void debug(CharSequence message) performs JndiLookup
}
```

Similar to a checked exception, because `JndiLogger` declares that `debug` performs a `JndiLookup`, our calling code must either handle that effect or declare that it, too, `performs` the lookup:

```java-effects
public class LogExample {
    public static void main(String[] args) performs JndiLookup {
        JndiLogger logger = LogManager.getJndiLogger();
        JndiLogger.debug("Here is a log message.");
    }
}
```

As with the `OpenFile` example, here we are just letting the runtime handle the effect, but we don't have to. Just like with an exception, we can handle an effect. For example, we can check that the effect is safe before we let it through:

```java-effects
public class LogExample {
    public static void main(String[] args) performs JndiLookup {
        JndiLogger logger = LogManager.getJndiLogger();
        try {
            JndiLogger.debug("Here is a log message.");
        } handle (JndiLookup lookup) {
            if (lookup.getName().contains("ldap")) {
                // Resume to caller without a lookup
                resume "[dangerous JNDI lookup]";
            } else {
                // This is a safe request, so let it through
                resume perform lookup;
            }
        }
    }
}
```

The `resume` keyword is similar to `return`: it returns control to the function that originally performed the effect, providing it with the specified value. In this example, we either provide a hard-coded string *or* we provide the result of performing the lookup, depending on whether the JNDI lookup attempts to use LDAP (in the real world we'd want a more complete check, obviously).

Alternatively, we can also handle the effect in all cases, in which case we don't have to declare any `performs` on `main`:

```java-effects
public class LogExample {
    public static void main(String[] args) {
        JndiLogger logger = LogManager.getJndiLogger();
        try {
            JndiLogger.debug("Here is a log message.");
        } handle (JndiLookup lookup) {
            resume "[no JNDI lookups allowed]";
        }
    }
}
```

Similar to capabilities, an effect system encodes security-sensitive side effects in the function signatures, which makes it easy to identify when code is doing something unexpected that we should take note of.

# Current implementations

There are already several languages that implement these concepts.

[Koka](https://koka-lang.github.io/koka/doc/index.html), by Microsoft, is built around effect handlers. [OCaml 5.0 got support for effects](https://discuss.ocaml.org/t/multicore-ocaml-september-2021-effect-handlers-will-be-in-ocaml-5-0/8554), with more improvements coming in future versions.

The [E language](http://wiki.erights.org/wiki/Main_Page) is an Object Oriented language built around capabilities. Their site also has a bunch of other resources for learning more about capabilities, including this example of [a simple digital currency](http://www.erights.org/elib/capability/ode/ode-capabilities.html).

# Summary

The current software supply chain situation is not sustainable: as Ken Thompson observed in the 80s, we cannot reliably expect to visually audit even a small amount of code, and the amount of open source code we use isn't small. Going forward, programming languages will need to adopt new features that make it easy to see security flaws early in the development process.

Capabilities and effect systems are two such systems that could be used to create more security programming languages. I've provided some simple examples of how these might be implemented in a language like Java. There are many flaws with these solutions—I intend them primarily to illustrate that implementing these features is both possible and could be added in a way that is in line with existing idioms.

Successfully implementing either approach in an existing language would require breaking changes, but the security improvements would be well worth the migration difficulties. We can't keep going the way we are, and both capabilities and effects offer a compelling path forward for greater security in our programming languages.

[^1]: [See this article for background on Log4Shell and JNDI.](https://www.shiftleft.io/blog/log4shell-jndi-injection-via-attackable-log4j/) For this post, it's sufficient to know that JNDI is a Java feature that allows executing remote code, and Log4J intentionally allowed using JNDI when logging, which created the vulnerability known as Log4Shell.

[^2]: [https://www.veracode.com/blog/research/exploiting-jndi-injections-java](https://www.veracode.com/blog/research/exploiting-jndi-injections-java)

[^3]: [http://web.archive.org/web/20211127190733/https://logging.apache.org/log4j/2.x/manual/lookups.html](http://web.archive.org/web/20211127190733/https://logging.apache.org/log4j/2.x/manual/lookups.html)

[^4]: [https://dl.acm.org/doi/pdf/10.1145/358198.358210](https://dl.acm.org/doi/pdf/10.1145/358198.358210)


[^5]: [https://en.wikipedia.org/wiki/Object-capability_model](https://en.wikipedia.org/wiki/Object-capability_model)

[^6]: [https://docs.scala-lang.org/scala3/reference/contextual/using-clauses.html](https://docs.scala-lang.org/scala3/reference/contextual/using-clauses.html)
