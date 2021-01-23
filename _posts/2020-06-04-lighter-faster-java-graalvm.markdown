---
layout: post
title:  "Lighter & Faster Java with GraalVM"
date:   2020-06-04 21:13:27 -0500
categories: GraalVM Native-Image Java
---

<h1>{{ "What is GraalVM?" }}</h1>

GraalVM is a Java VM and JDK based on HotSpot / OpenJDK, implemented in Java. It offers a comprehensive ecosystem supporting a large set of languages (Java and other JVM-based languages, JavaScript, Ruby, Python, R, Web Assembly, C/C++ and other LLVM-based languages). With ahead-of-time compilation feature, it compiles applications into smaller self-contained native images which results in faster startup and lower memory footprint and we can still use it as Just-in-time compiler. The native-image tool also packages the SubstrateVM (SVM) into the image to provide functionalities like garbage collection. With these benefits. it’s definitely a top choice for containers, for applications running on cloud, specially Lambda.

Read more about Ahead-of-time compilation in my article <a href="https://techsikandar.github.io/ahead-of-time/just-in-time/aot/jit/java/2020/03/17/just-in-time-vs-ahead-of-time.html" target="_blank">here</a>.

<h1>{{ "Some limitations" }}</h1>

There are some limitations, related to native-image capabilities, which can be resolved at image build time by providing the configuration files. For example:

<ul>
<li><b>Dynamic class loading:</b> The calls like <i>Class.forName("someclass")</i> must have <i>"someclass"</i> defined in the configuration so that it can be found during image build.</li>
<li><b>Reflection:</b> native-image statically analyze the code to resolve all such calls. Calls which are not resolved must be specified in image build configuration.</li>
</ul>

More can be found <a href="https://github.com/oracle/graal/blob/master/substratevm/Limitations.md" target="_blank">here</a>.

<h1>{{ "GraalVM support in other frameworks" }}</h1>

GraalVM is supported by many microservices frameworks like <a href="https://helidon.io/#/" target="_blank">helidon</a>, <a href="https://quarkus.io/" target="_blank">quarkus</a> and <a href="https://micronaut.io/" target="_blank">micronaut</a>. You can also developed GraalVM based microservices in Spring. Refer to the sample code <a href="https://github.com/graalvm/graalvm-demos/tree/master/spring-r" target="_blank">here</a>.

<h1>{{ "Let's play with it" }}</h1>

<b> {{ "Download & Install" }} </b><br>

Download the GraalVM from <a href="https://github.com/graalvm/graalvm-ce-builds/releases" target="_blank">here</a>. Please make sure to download the correct version based upon your OS. I am using Linux and i downloaded `graalvm-ce-java11-linux-amd64–20.1.0.tar.gz`.

{% highlight ruby %}
wget https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.1.0/graalvm-ce-java11-linux-amd64-20.1.0.tar.gz
{% endhighlight %}

Now extract the tar file:

{% highlight ruby %}
tar -xvzf graalvm-ce-java11-linux-amd64-20.1.0.tar.gz
{% endhighlight %}

And then set paths:

{% highlight ruby %}
export PATH=<path to GraalVM>/bin:$PATH export JAVA_HOME=<path to GraalVM>
{% endhighlight %}

You may run `java -version` to check if its pointing to right java version.

<b> {{ "Install Native Image" }} </b><br>

{% highlight ruby %}
gu install native-image yum install libstdc++-static yum install zlib-devel.x86_64
{% endhighlight %}

You may or may not require `zlib-devel.x86_64`. Refer to native image installation instructions <a href="https://www.graalvm.org/reference-manual/native-image/" target="_blank">here</a> for more details. Remember to install all prerequisites. You should be all set now.

<b> {{ "Run our favorite 'Hello World' program" }} </b><br>

Create a simple HelloWorld.java program.

{% highlight ruby %}
public class HelloWorld { 
    public static void main(String[] args) { 
        System.out.println("Hello, World!"); 
    } 
}
{% endhighlight %}

Compile and run the program and capture the time.

{% highlight ruby %}
$ javac HelloWorld.java 
$ time java HelloWorld 
$ Hello, World! real0m0.117s user0m0.099s sys0m0.012s
{% endhighlight %}

Now, let’s create the native image of the HelloWorld program

{% highlight ruby %}
$ native-image HelloWorld
{% endhighlight %}

This will generate an executable named `helloworld`. You can simply execute like you execute any other executable and then it will execute the natively compiled HelloWorld.

{% highlight ruby %}
$ time ./helloworld 
$ Hello, World! real0m0.002s user0m0.002s sys0m0.000s
{% endhighlight %}

See the time difference.

<h1>{{ "Conclusion" }}</h1>

Its interesting to see how Java is continuously evolving & modernizing along with the ever changing technology landscape around us. It’s promising, specially in cloud.