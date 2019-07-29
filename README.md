# RESTful Microservice

**REST** stands for Representational State Transfer, which is an architectural style for developing web services. **Microservices** - also known as the microservice architecture - is an architectural style that structures an application as a collection of services that are. Highly maintainable and testable. Loosely coupled. Independently deployable. Organized around business capabilities.

A microservice, based on REST is called a **RESTful** microservice. REST is not dependent on any protocol, but almost every RESTful service uses HTTP as its underlying protocol. It's also very popular in modern API oriented application development.

In this article, I will be discussing 2 different technologies to implement a RESTful microservice and discuss the pros and cons of both approaches. Two technologies which I am going to use;
 1. SpringBoot with Apache Camel
 2. Ballerina

**SpringBoot** is an organized Java framework for building microservices-based on the Spring Dependency Injection framework with minimal code and configuration and allows packaging them into an isolated executable jar giving you production-ready microservices. **Apache Camel** is a routing engine which provides an implementation for almost all Enterprise Integration Patterns with many different components to facilitate integration with various systems.

**Ballerina** is a programming language to create network-distributed applications. It is not a DSL but it’s a full programming language with a rich set of general-purpose features which is optimized for “network distributed applications”

# What we’ll build
To understand how we can build a RESTful microservice, let’s consider a real-world use case of an order management scenario of an online retail application. Following list covers all functionalities in an order management system.
* Create an order
* Retrieve an order
* Update an order
* Delete an order

The following diagram illustrates all the required functionality of the Order Management RESTful microservice that we are going to build.

![order mgt](https://github.com/lakwarus/ballerina-camel-springboot-restful-service/blob/master/images/order-mgt.png)

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

![project](https://github.com/lakwarus/ballerina-camel-springboot-restful-service/blob/master/images/project.png)

After clicking Next, you would get Spring starter dependency window. Lookup for Web and Camel and select Web and Apache Camel option respectively like on below screen and click Finish.

![Camel-dependancy](https://github.com/lakwarus/ballerina-camel-springboot-restful-service/blob/master/images/Camel-dependancy.png)

If everything went fine you should end up with the below project in the workspace.

![source-structure](https://github.com/lakwarus/ballerina-camel-springboot-restful-service/blob/master/images/source-structure.png)

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

## Implementation

When we initialized the project, Spring Boot has created SpringbootCamelRestdslApiApplication bootstrap class with main() method to run the application. We will now inject Camel Servlet and Camel route to this class.

```java
package com.lakwarus.sample.pojo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootCamelRestdslApplication {

	public SpringbootCamelRestdslApplication() {
	}
```

Apache Camel offers a REST styled DSL which can be used with Java or XML. The intention is to allow end-users to define REST services using a REST-style with verbs such as GET, POST, DELETE etc. In Camel we can use multiple components to implement Rest DSL. The following list is the Camel components which support the Rest DSL.
- camel-coap
- camel-netty-http (also supports Swagger Java)
- camel-netty4-http (also supports Swagger Java)
- camel-jetty (also supports Swagger Java)
- camel-restlet (also supports Swagger Java)
- camel-servlet (also supports Swagger Java)
- camel-spark-rest (also supports Swagger Java from Camel 2.17)
- camel-undertow (also supports Swagger Java from Camel 2.17)

I have used camel-servlet REST DSL which acts as façade for REST endpoints.

Camel has a way to set CamelServlet registration from Camel version 2.19.0. It is default registered at “/camel” endpoint which you can optionally overwrite. I have overwritten it to /*. Edit src/main/resources/application.properties and add the below property.

```
camel.component.servlet.mapping.context-path=/*
```

To configure REST DSL to use the Servlet component implementation we have used restConfiguration().component(“servlet”). When we add bindingMode(RestBindingMode.json) we tell Spring to format the incoming and outgoing POJOs to JSON format. 

```java
restConfiguration().component("servlet").bindingMode(RestBindingMode.json);

            onException(Exception.class).handled(true).process(new Processor() {

                public void process(Exchange exchange) throws Exception {
                    Exception ex = (Exception) exchange.getProperty(Exchange.EXCEPTION_CAUGHT);

                    if (ex instanceof JsonEOFException) {
                        Status status = new Status();
                        status.setOrderId("null");
                        status.setStatus("Malformed JSON recevied");

                        // Create response message.
                        exchange.getOut().setBody(status, Status.class);
                    } else {
                        Status status = new Status();
                        status.setOrderId("null");
                        status.setStatus(" Error occured while proceesing the request !!!");

                        // Create response message.
                        exchange.getOut().setBody(status, Status.class);
                    }

                }
            });
```

I have created two resources classes (Order.class and Status.class) which use to format incoming and outgoing POJOs to JSON format.  

```java
package com.lakwarus.sample.pojo;


public class Order {

	private String id;
	private String name;
	private String description;
	

	public String getId() {
		return id;
	}

	public void setId(String id){
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getDescription() {
		return this.description;
	}

	public void setDescription(String description) {
		this.description = description;
	}

}
```

```java
package com.lakwarus.sample.pojo;

public class Status {
	private String status;
	private String orderId;
	
	public String getStatus() {
		return status;
	}

	public void setStatus(String status) {
		this.status = status;
	}

	public String getOrderId() {
		return orderId;
	}

	public void setOrderId(String orderId) {
		this.orderId = orderId;
	}

}
```

TODO - explain OnException handling

The complete main() application class will now look at how we can add the camel route. You can also define Camel Route in a separate class with @Component annotation.

Let's look at the implementation of create order functionality first.

```java
// Resource that handles the HTTP POST requests that are directed to the path
// '/order' to create a new Order.            rest("/ordermgt").consumes("application/json").post("/order").type(Order.class).to("direct:addOrder");

            from("direct:addOrder").doTry().process(new Processor() {

                @Override
                public void process(Exchange exchange) throws Exception {

                    Order order = exchange.getIn().getBody(Order.class);
                    String orderId = order.getId();
                    if (orderId == null) {
                        throw new OrderValidationException();
                    }
                    objectmap.put(orderId, order);

                    Status status = new Status();
                    status.setOrderId(orderId);
                    status.setStatus("Order Created!");

                    // Create response message.
                    exchange.getOut().setBody(status, Status.class);

                    // Set 201 Created status code in the response message.
                    exchange.getOut().setHeader(Exchange.HTTP_RESPONSE_CODE, 201);
                    // Set 'Location' header in the response message.
                    // This can be used by the client to locate the newly added order.
                    exchange.getOut().setHeader("Location", "http://localhost:8080/ordermgt/order/" + orderId);
                }
            }).doCatch(OrderValidationException.class).process(new Processor() {

                public void process(Exchange exchange) throws Exception {

                    Status status = new Status();
                    status.setOrderId("null");
                    status.setStatus("Order Id is Null !!");

                    // Create response message.
                    exchange.getOut().setBody(status, Status.class);
                    exchange.getOut().setHeader(Exchange.HTTP_RESPONSE_CODE, 400);

                }
            }).log("AddOrder error : Invalid JSON recevied!").doCatch(Exception.class).process(new Processor() {

                public void process(Exchange exchange) throws Exception {
                    if (exchange.getOut().getBody(Order.class) == null) {
                        Status status = new Status();
                        status.setOrderId("null");
                        status.setStatus("JSON payload is empty!");

                        // Create response message.
                        exchange.getOut().setBody(status, Status.class);
                        exchange.getOut().setHeader(Exchange.HTTP_RESPONSE_CODE, 400);
                    }

                }
            }).log("AddOrder error : JSON payload is empty!");
```

We can implement the business logic of each resource depending on your requirements. For simplicity, I used an in-memory map to keep all the order details.

TODO - explain Camel route and error handling

Following code, block shows how I implemented get order functionality.

```java
	    // Resource that handles the HTTP GET requests that are directed to a specific
            // order using path '/order/<orderId>'.
            rest("/ordermgt").get("/order/{orderId}").to("direct:getOrder");

            from("direct:getOrder").doTry().process(new Processor() {

                @Override
                public void process(Exchange exchange) {

                    // Find the requested order from the map and retrieve it in JSON format.
                    String orderId = exchange.getIn().getHeader("orderId", String.class);
                    Order order = objectmap.get(orderId);

                    if (order == null) {

                        Status status = new Status();
                        status.setOrderId(orderId);
                        status.setStatus("Get order error : " + orderId + " cannot be found!");

                        exchange.getOut().setHeader(Exchange.HTTP_RESPONSE_CODE, 400);
                    } else {

                        exchange.getOut().setBody(order, Order.class);
                        exchange.getOut().setHeader(Exchange.HTTP_RESPONSE_CODE, 200);
                    }

                }
            }).doCatch(Exception.class).process(new Processor() {

                public void process(Exchange exchange) throws Exception {

                    Status status = new Status();
                    status.setOrderId("null");
                    status.setStatus("Internal server error!");

                    exchange.getOut().setHeader(Exchange.HTTP_RESPONSE_CODE, 500);

                }
            }).log("Get order error : error while processing getOrder");
```

In the same way, I have implemented update-order and the delete-order functionalities and full source code of the application can be found here. 

## Testing
We can run the RESTful microservice that we developed above, in your local environment. 

Open SpringbootCamelRestdslApiApplication class and Run as Java Application or SpringBoot Application. Check the console tab for application build status. 

To test the functionality of the orderMgt RESTFul microservice, send HTTP requests for each order management operation. Following are sample cURL commands that you can use to test each operation of the order management service.

#### Create order

```bash
$ curl -v -X POST -d \
    '{ "ID": "100500", "Name": "XYZ", "Description": "Sample order."}' \
    "http://localhost:8080/ordermgt/order" -H "Content-Type:application/json"

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> POST /ordermgt/order HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 64
> 
* upload completely sent off: 64 out of 64 bytes
< HTTP/1.1 201 
< Location: http://localhost:8080/ordermgt/order/100500
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Mon, 29 Jul 2019 19:00:20 GMT
< 
* Connection #0 to host localhost left intact
{"status":"Order Created!","orderId":"100500"}
```

#### Retrieve Order

```bash
$ curl -v "http://localhost:9090/ordermgt/order/100500"

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /ordermgt/order/100500 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Mon, 29 Jul 2019 19:02:04 GMT
< 
* Connection #0 to host localhost left intact
{"id":"100500","name":"XYZ","Description":"Sample order."}
```

#### Update Order

```bash
$ curl -X PUT -d '{ "ID": "100500", "Name": "XYZ", "Updated Description": "Sample order."}'' \
    "http://localhost:8080/ordermgt/order/100500" -H "Content-Type:application/json"

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> PUT /ordermgt/order/100500 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 71
> 
* upload completely sent off: 71 out of 71 bytes
< HTTP/1.1 200 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Mon, 29 Jul 2019 19:06:47 GMT
< 
* Connection #0 to host localhost left intact
{"id":"100500","name":"XYZ","Updated Description":"Sample order."}
```

#### Delete Order

```bash
$ curl -X DELETE "http://localhost:9090/ordermgt/order/100500"

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> DELETE /ordermgt/order/100500 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Mon, 29 Jul 2019 19:08:13 GMT
< 
* Connection #0 to host localhost left intact
{"status":"Order deleted!","orderId":"100500"}
```

## Deployment
TODO - Camek-K

# Ballerina

## Prerequisites

- Ballerina Distribution
- A Text Editor or an IDE. I have used VSCode with Ballerina VSCode plugin

### Optional requirements
- Docker
- Kubernetes

## Setting up the project

## Implementation

## Testing


## Deployment
# Summary
TODO 
- Explain the Spring+Camel pain points of readability
- Explain the Spring+Camel pain points of exception handling
- Explain the Spring+Camel pain points of dependency management
- Explain the Ballerina advantage of @Docker and @Kubernetes annotation
- Explain the Ballerina advantage of JSON handling
- Explain the Ballerina advantage of error handling
- Explain the Ballerina advantage of sequence diagram
- Explain Spring+Camel advantage/pain of multiple chose for REST DSL
- Explain Spring+Camel advantage of available contents

# References
- http://www.javaoutofbounds.com/apache-camel-springboot-rest-api-example/
- https://www.javainuse.com/camel/camelException
- https://camel.apache.org/rest-dsl.html
- https://ballerina.io/learn/by-guide/restful-service


