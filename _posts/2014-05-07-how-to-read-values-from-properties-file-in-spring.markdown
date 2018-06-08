---
layout: post
title: Read Property Value from Properties File in Spring
date: '2014-05-07 10:02:09'
tags:
- spring
---

The Java properties file can be used to store project configurations or settings.

<strong> constant.properties</strong>
{% highlight java %}
ui.defaultIconUrl=graph.facebook.com/ek.pranav/picture
{% endhighlight %}

Environment - Interface representing the environment in which the current application is running.

@Configuration - Configuration classes are candidates for component scanning (typically using Spring XML's &lt;context:component-scan/&gt; element) and therefore may also take advantage of @Autowired/@Inject at the field and method level (but not at the constructor level).

@PropertySource - Adds property sources to the enclosing Environment.

&nbsp;

The values in the properties can be accessed by injecting Environment object to the configuration class.

<strong> ConstantPropertyInjector.java</strong>
{% highlight java %}
@Configuration
@PropertySource("classpath:META-INF/constant.properties")
public class ConstantPropertyInjector {

@Autowired Environment env;

   @Bean
   public ConstantImageHolder getConstantProperty(){
     ConstantImageHolder constant = new ConstantImageHolder ();
     constant.setDefaultIcon(env.getProperty("ui.defaultIconUrl"));
     return constant;
   }

}
{% endhighlight %}

&nbsp;
