# Spring Data DynamoDB ( Work in Progress ) #

The primary goal of the [Spring Data](http://www.springsource.org/spring-data) project is to make it easier to build Spring-powered applications that use data access technologies. This module deals with enhanced support for Amazon DynamoDB based data access layers.

## Supported Features ##

* Implementation of CRUD methods for DynamoDB Entities
* Dynamic query generation from query method names  (Only a limited number of keywords and comparison operators currently supported)
* Possibility to integrate custom repository code
* Easy Spring annotation based integration

## Demo application ##

For a demo of spring-data-dynamodb, using spring-data-rest to showcase DynamoDB repositories exposed with REST,
please see <a href="https://github.com/michaellavelle/spring-data-dynamodb-demo">spring-data-dynamodb-demo

## Quick Start ##

Download the jar though Maven:


```xml
<repository>
	<id>opensourceagility-snapshots</id>
	<url>http://repo.opensourceagility.com/snapshots</url
</repository>
```

```xml
<dependency>
  <groupId>org.socialsignin</groupId>
  <artifactId>spring-data-dynamodb</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>
```

Setup DynamoDB configuration as well as enabling Spring Data DynamoDB repository support.

```java
@Configuration
@EnableDynamoDBRepositories(basePackages = "com.acme.repositories")
public class DynamoDBConfig {

	@Value("${amazon.dynamodb.endpoint}")
	private String amazonDynamoDBEndpoint;

	@Value("${amazon.aws.accesskey}")
	private String amazonAWSAccessKey;

	@Value("${amazon.aws.secretkey}")
	private String amazonAWSSecretKey;

	@Bean
	public AmazonDynamoDB amazonDynamoDB() {
		AmazonDynamoDB amazonDynamoDB = new AmazonDynamoDBClient(
				amazonAWSCredentials());
		if (StringUtils.isNotEmpty(amazonDynamoDBEndpoint)) {
			amazonDynamoDB.setEndpoint(amazonDynamoDBEndpoint);
		}
		return amazonDynamoDB;
	}

	@Bean
	public AWSCredentials amazonAWSCredentials() {
		return new BasicAWSCredentials(amazonAWSAccessKey, amazonAWSSecretKey);
	}

}
```

or in xml...

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://socialsignin.org/schema/data/dynamodb
                           http://socialsignin.org/schema/data/dynamodb/spring-dynamodb.xsd">

  <bean id="amazonDynamoDB" class="com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient">
    <constructor-arg ref="amazonAWSCredentials" />
    <property name="endpoint" value="${amazon.dynamodb.endpoint}" />
  </bean>
  
  <bean id="amazonAWSCredentials" class="com.amazonaws.auth.BasicAWSCredentials">
    <constructor-arg value="${amazon.aws.accesskey}" />
    <constructor-arg value="${amazon.aws.secretkey}" />
  </bean>
  
  <dynamodb:repositories base-package="com.acme.repositories" />
  
</beans>

```xml

Create a DynamoDB hash-key only table in AWS console, with table name 'User' and with hash key attribute name "id"

Create a DynamoDB entity for this table:

```java
@DynamoDBTable(tableName = "User")
public class User {

  private String id;
  private String firstName;
  private String lastName;

  @DynamoDBHashKey
  @DynamoDBAutoGeneratedKey 
  public String getId()
  {
	return id;
  }

  @DynamoDBAttribute
  public String getFirstName()
  {
	return firstName;
  }

  @DynamoDBAttribute
  public String getLastName()
  {
	return lastName;
  }
       
  // setters, default constructor and firstname/lastname constructor
}
```

Create a repository interface in `com.acme.repositories`:

```java
public interface UserRepository extends CrudRepository<User, String> {
  List<User> findByLastName(String lastName);
}
```

Write a test client

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:your-config-file.xml")
public class UserRepositoryIntegrationTest {
     
  @Autowired UserRepository repository;
     
  @Test
  public void sampleTestCase() {
    User dave = new User("Dave", "Matthews");
    repository.save(user);
         
    User carter = new User("Carter", "Beauford");
    repository.save(carter);
         
    List<User> result = repository.findByLastName("Matthews");
    assertThat(result.size(), is(1));
    assertThat(result, hasItem(dave));
  }
}
```


