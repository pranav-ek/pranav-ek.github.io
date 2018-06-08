---
layout: post
title: Using Hibernate SessionFactory in Spring Boot App
date: '2016-09-20 10:11:20'
---

To autowire Hibernate SessionFactory in the Spring Boot application, the *SpringSessionContext* - implementation of the Hibernate's *[CurrentSessionContext](//docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/context/CurrentSessionContext.html)* interface - can be used. 


#### 1. Add entry in *application.properties* file
Specify *spring.jpa.properties.hibernate.current_session_context_class* property with *[SpringSessionContext](//docs.spring.io/spring/docs/current/javadoc-api/org/springframework/orm/hibernate3/SpringSessionContext.html)* class as the value.

{% highlight java %}

spring.jpa.properties.hibernate.current_session_context_class=org.springframework.orm.hibernate4.SpringSessionContext
{% endhighlight %}
#### 2. Bean definition
Create a bean that will attach *[HibernateJpaSessionFactoryBean](//docs.spring.io/spring/docs/current/javadoc-api/org/springframework/orm/jpa/vendor/HibernateJpaSessionFactoryBean.html)* to the container.

{% highlight java %}
@Configuration
public class DBConfig {

	@Bean
	public HibernateJpaSessionFactoryBean sessionFactory() {
	    return new HibernateJpaSessionFactoryBean();
	}
	
}
{% endhighlight %}

#### 3. Autowire the *SessionFactory* on the data layer

{% highlight java %}
@Repository
public class OrderDaoImpl implements OrderDao {
	
	@Autowired
	private SessionFactory sessionFactory;
		
	// rest of code removed for brevity

}
{% endhighlight %}
