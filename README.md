# RESTful Microservice

**REST** stands for Representational State Transfer, which is an architectural style for developing web services. **Microservices** - also known as the microservice architecture - is an architectural style that structures an application as a collection of services that are. Highly maintainable and testable. Loosely coupled. Independently deployable. Organized around business capabilities.

A microservice, based on REST is called a **RESTful** microservice. REST is not dependent on any protocol, but almost every RESTful service uses HTTP as its underlying protocol. It's also very popular in modern API oriented application development.

In this article, I will be discussing 2 different technologies to implement a RESTful microservice and discuss the pros and cons of both approaches. Two technologies which I am going to use;
 - SpringBoot with Apache Camel
 - Ballerina

**SpringBoot** is an organized Java framework for building microservices-based on the Spring Dependency Injection framework with minimal code and configuration and allows packaging them into an isolated executable jar giving you production-ready microservices. **Apache Camel** is a routing engine which provides an implementation for almost all Enterprise Integration Patterns with many different components to facilitate integration with various systems.

**Ballerina** is a programming language to create network-distributed applications. It is not a DSL but it’s a full programming language with a rich set of general-purpose features which is optimized for “network distributed applications”

# What we’ll build
To understand how we can build a RESTful microservice, let’s consider a real-world use case of an order management scenario of an online retail application. Following list covers all functionalities in an order management system.
- Create an order
- Retrieve an order
- Update an order
- Delete an order

The following diagram illustrates all the required functionality of the Order Management RESTful microservice that we are going to build.

# SpringBoot with Apache Camel
Here we will create a Camel REST microservice using REST DSL, further we will use Camel Servlet to expose the REST API.

## Prerequisites
- JDK 1.8+
- Maven 3.2+
- Your favorite IDE, I have used Spring Tool Suite (STS)

### Optional requirements
- Docker
- Kubernetes
- TODO - Camel-K?

## Setting up the project
Start with STS, first we need to create a maven project with dependencies. Project details I have used;
- GroupID: com.lakwarus
- ArtifactID: springboot-camel-restdsl
- Package: com.lakwarus.sample.pojo

TODO - image

After clicking Next, you would get Spring starter dependency window. Lookup for Web and Camel and select Web and Apache Camel option respectively like on below screen and click Finish.

TODO - image

If everything went fine you should end up with the below project in the workspace.

TODO - image

Now we need to add all the required dependencies into the pom.xml

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.apache.camel</groupId>
			<artifactId>camel-spring-boot-starter</artifactId>
			<version>2.24.0</version>
		</dependency>
	  	<dependency>
			<groupId>org.apache.camel</groupId>
			<artifactId>camel-servlet-starter</artifactId>
			<version>2.24.0</version>
		</dependency>
		<dependency>
			<groupId>org.apache.camel</groupId>
			<artifactId>camel-jackson</artifactId>
			<version>2.24.0</version>
		</dependency>
		<dependency>
 			<groupId>org.apache.camel</groupId>
  			<artifactId>camel-jsonpath</artifactId>
			<version>2.24.0</version>
		</dependency>
	</dependencies>
```
