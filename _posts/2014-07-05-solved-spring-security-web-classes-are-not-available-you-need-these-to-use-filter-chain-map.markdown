---
layout: post
title: "[Solved] spring-security-web classes are not available. You need these to
  use filter-chain-map"
date: '2014-07-05 18:21:34'
---

Recently I downloaded the latest <a title="STS" href="http://spring.io/tools" target="_blank">Spring Tool Suite (STS)</a> with Java 8 SE support. My intention was to give it a try to Stream API on my personal project. An error was shown on the spring security configuration XML once the [Maven](//maven.apache.org) project is imported to the STS.

The STS displayed following configuration error:
“ spring-security-web classes are not available. You need these to use &lt;filter- chain-map&gt;”

Searched about the error and this is what I found.

FilterChainProxy class (org.springframework.security.web.FilterChainProxy) of spring security artifact  imports few javax.servlet classes. The [pom.xml](//maven.apache.org/pom.html) didn't have servlet library dependency in it.

Added following <a title="javax.servlet" href="http://mvnrepository.com/artifact/javax.servlet/servlet-api" target="_blank">servlet dependency</a> to it.

```language-xml
<dependency>
     <groupId>javax.servlet</groupId>
     <artifactId>servlet-api</artifactId>
     <version>{version}</version>
 </dependency>
```

After maven install, the error was  gone, so I decided to initiate the app.

The console displayed following exception  long before I could see the index page:

"ServletDispatcher cannot be cast to Javax.servlet.Servlet exception"

Adding servlet-api artifact will include servlet JAR along with the container specific servlet libraries, which would load different servlet API class-loaders at run time and causes the above exception.  The servlet artifact is only needed at compilation in this case, so setting the scope to 'provided' will resolve the issue.

About dependency scope from <a title="maven dependency scope" href="http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope" target="_blank">Maven </a>doc:
<ul>
	<li><b>compile</b>
This is the default scope, used if none is specified. Compile dependencies are available in all classpaths of a project. Furthermore, those dependencies are propagated to dependent projects.</li>
	<li><b>provided</b>
This is much like <tt>compile</tt>, but indicates you expect the JDK or a container to provide the dependency at runtime. For example, when building a web application for the Java Enterprise Edition, you would set the dependency on the Servlet API and related Java EE APIs to scope <tt>provided</tt> because the web container provides those classes. This scope is only available on the compilation and test classpath, and is not transitive.</li>
</ul>
&nbsp;

The final dependency definition for javax.servlet with 'provided' scope
```language-xml
<dependency>
     <groupId>javax.servlet</groupId>
     <artifactId>servlet-api</artifactId>
     <version>3.0.1</version>
     <scope>provided</scope>
 </dependency>
```
&nbsp;