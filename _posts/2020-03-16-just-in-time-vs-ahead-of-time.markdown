---
layout: post
title:  "Just in time & Ahead of time Compilation"
date:   2020-03-16 21:13:27 -0500
categories: java
---

<h1>{{ "So what is Just in time?" }}</h1>

Languages are traditionally either compiled or interpreted into native machine code. But for Java to fulfill it’s promise of `Write Once, Run Anywhere`, it kinda does both. Java first converts the source code into a byte code which is then converted into native machine code. The byte-code to machine-code conversion happens on the spot at runtime that’s why the process is called `Just in time`. It kicks in when Java program is launched. JIT Compiler is a feature of Java Runtime which performs this conversion.

<h1>{{ "How JIT improves the Java runtime performance?" }}</h1>

I will divide a Java application into two parts. There will always be parts of the code which will be accessed more frequently then others. Java employs different strategies for both:

<ul>
<li><i>Infrequently access code:</i> If the code is rarely accessed, infrequently accessed or accessed only once then it directly interprets the byte-code to machine-code because JIT compilation could be more expensive and not worth it.</li>
<li><i>Frequently used:</i> Performance of an application depends upon how fast Java can execute the frequently used code. These are also called hot spots. So, it makes more sense for JIT compiler to compile these codes from byte-code to machine-code and store the resultant machine-code into code cache for subsequent execution. It then keeps on optimizing the execution with every cycle.</li>
</ul>

JIT compilers itself has two types:
<ul>
<li><i>Client Compiler (C1):</i> The client compiler starts compiling as soon as the application starts and it compiles aggressively. As a result, initially, application execution looks faster. Client Compiler will be your choice if you are looking for quick application startup time. This can be enabled with <i>-client</i> switch. </li>
<li><i>Server Compiler (C2):</i> The Server compiler waits and analyze the execution application behavior and then compiles with the best possible optimization. As a result, it will start slow but it will improve over the period of time. This can be enabled with <i>-server</i> switch.</li>
</ul>

Or alternatively choose tiered compilation, which can be enabled with a switch `-XX:+TieredCompilation`, which combines both approaches. Client compiler compiles first, prepares instruction sets for advanced optimization for server compiler and then server compiler kicks in. It’s ideal for cases when you have long running application.

<h1>{{ "What is Ahead of Time Compilation?" }}</h1>

Actually there is an inherent issue with the way JIT compiler works. In large scale and complex Java applications, there may be many libraries, classes and methods which are never invoked and when they are invoked they will trigger lots of interpretations tasks ultimately leading to performance bottlenecks. <i>Java AOT compiler compiles classes to native machine code before launching the programs.</i> That’s why they are called Ahead of Time, and there is no need for Just in Time compilation.

Java has a command jaotc which can perform the Ahead of Time compilation for you.

Generate AOT library: `jaotc –output Test.so Test.class`<br>
Invoke the program with AOT library: `java –XX:AOTLibrary=./Test.so Test`

<h1>{{ "When to use AOT?" }}</h1>

Ahead of Time compilation may not always bring performance improvement to your Java applications. Consider these points before making a choice:

<ul>
<li>Identify the classes/method that you think should be compiled ahead of time. Experiment with AOT and measure their performance against JIT.</li>
<li>Large & complex applications may benefit from AOT but again it’s not guaranteed. Measure the performance as mentioned above.</li>
</ul>

<h1>{{ "Conclusion" }}</h1>

Just in Time compilation has an edge because it has the information about it’s runtime. AOT does not have this luxury. That’s the reason why AOT must be executed on the same system on which it’s code will be executed. Just in Time compilation learns, evolve, optimizes over the time and it has more room for improvement. That’s not the case with AOT.
