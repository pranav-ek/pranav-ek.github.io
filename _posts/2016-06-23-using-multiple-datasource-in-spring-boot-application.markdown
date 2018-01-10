---
layout: post
title: Using  Multiple DataSource In Spring Boot App
date: '2016-06-23 13:08:44'
---

#### 1. Overview
Spring Boot will load properties from the [application.properties](//docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) file which resides in the application class path and add them to the Spring [Environment](//docs.spring.io/spring/docs/4.0.1.RELEASE/javadoc-api/org/springframework/core/env/Environment.html). 

This post shows how to create multiple JdbcTemplate instances which are bounded to different databases specified in the properties files.   

#### 2. The *application.properties*
Sample properties file located at [src/main/resources](//maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)
{% highlight java %}
#DB
spring.db_mysql.driverClassName=com.mysql.jdbc.Driver
spring.db_mysql.url=jdbc:mysql://localhost:3306/test
spring.db_mysql.username=root_branch_leaf
spring.db_mysql.password=hard_to_guess
 
spring.db_sqlite.driverClassName=org.sqlite.JDBC
spring.db_sqlite.url=jdbc:sqlite:test.db
{% endhighlight %}
#### 3. Configuration for DB
Configure [DataSource](//docs.oracle.com/javase/7/docs/api/javax/sql/DataSource.html) bean in the Spring configuration class named *DbConfig*. Using @ConfigurationProperties we could bind the namespace properties to the given DataSource bean.

{% highlight java %}
@Configuration
public class DbConfig {

	@Bean(name = "mysqlDb")
	@Primary
	@ConfigurationProperties(prefix="spring.db_mysql") 
	public DataSource mysqlDataSource() {
		return DataSourceBuilder.create().build();
	}
	
	@Bean(name = "sqliteDb")
	@ConfigurationProperties(prefix="spring.db_sqlite") 
	public DataSource sqliteDataSource() {
		return DataSourceBuilder.create().build();
	}
	
	@Bean(name = "mysql")
    public JdbcTemplate mysqlJdbcTemplate(@Qualifier("mysqlDb") DataSource mysqlDb) { 
		return new JdbcTemplate(mysqlDb); 
    } 
		
	@Bean(name = "sqlite") 
    public JdbcTemplate sqliteJdbcTemplate(@Qualifier("sqliteDb") DataSource sqliteDb) { 
        return new JdbcTemplate(sqliteDb); 
    } 
	
}
{% endhighlight %}

#### 5. Create an interface for the DAO 
Write an abstract method inside the *MedicaidDao* interface
{% highlight java %}
public interface MedicaidDao {
	Map<String, String> findMetaDataOfTemplateBxp();
}
{% endhighlight %}

#### 6. Implement the DAO interface
Autowire the [JdbcTemplate](//docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) to a field and use [@Qualifier](//docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Qualifier.html) annotation to specify which bean should be injected to it.

{% highlight java %}
@Repository
public class MedicaidDaoImpl implements MedicaidDao {

	static final Logger LOG = LoggerFactory.getLogger(MedicaidDaoImpl.class);

	@Autowired
	@Qualifier("mysql")
	private JdbcTemplate jdbcTemplate;

	@Autowired
	@Qualifier("sqlite")
	private JdbcTemplate sqliteJdbcTemplate;

	@Override
	public Map<String,String> findMetaDataOfTemplateBxp() {
		Map<String,String> bxpMetaDataMap =  new HashMap<String,String>();
		jdbcTemplate.query("SELECT column_name, column_type FROM INFORMATION_SCHEMA.columns where table_schema=? and table_name=?", (ps) -> {
			ps.setString(1,"test");
			ps.setString(2,"bxps");
		},(rs) -> {
			while(rs.next()){
				bxpMetaDataMap.put(rs.getString("column_name"),rs.getString("column_type")); 				
			}
		});
		
		return bxpMetaDataMap;
	}
}
{% endhighlight %}
#### 7. Conclusion
[DataSourceBuilder](//docs.spring.io/autorepo/docs/spring-boot/1.2.7.RELEASE/api/org/springframework/boot/autoconfigure/jdbc/DataSourceBuilder.html) builds data source from the values specified in the properties file. 

Using the @Qualifier annotation we could specify the JdbcTemplate bean which should be injected in the DAO. Data will be fetched from the data source bounded to the bean.
