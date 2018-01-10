---
layout: post
title: Loading Multiple Version of the Same Class
date: '2016-09-19 13:13:02'
---

There comes a time we are forced to use multiple version of a library in the project. The Java system classloader will load a specific class only once. To load different implementation of a class reside in different versions of JAR file, we can make use of [java.net.URLClassLoader](//docs.oracle.com/javase/7/docs/api/java/net/URLClassLoader.html). 

Following is an example code of using URLClassLoader to load the class named HelloPrint from two different versions of Hello library.

#### The *ClassloaderMultipleVersion.java* 
{% highlight java %}
public class ClassloaderMultipleVersion {

	public static void main(String[] args) throws Exception {

		ClassLoader loaderA = new URLClassLoader(new URL[] { new File("Hellov1.jar")
																.toURI().toURL() });
		ClassLoader loaderB = new URLClassLoader(new URL[] { new File("Hellov2.jar")
																.toURI().toURL() });

		Class<?> classA = loaderA.loadClass("com.prnv.hello.HelloPrint");
		Class<?> classB = loaderB.loadClass("com.prnv.hello.HelloPrint");

		Method methodA = classA.getMethod("hello");
		Method methodB = classB.getMethod("hello");

		methodA.invoke(classA.newInstance());
		methodB.invoke(classB.newInstance());

	}
}

{% endhighlight %}

This example make use of [Java Reflection](//docs.oracle.com/javase/tutorial/reflect/) to invoke the method. 

