# RESTful Microservices : Apache Camel+SpringBoot Vs Ballerina

**REST** stands for Representational State Transfer, which is an architectural style for developing web services. **Microservices** - also known as the microservice architecture - is an architectural style that structures an application as a collection of services that are. Highly maintainable and testable. Loosely coupled. Independently deployable. Organized around business capabilities.

A microservice, based on REST is called a **RESTful** microservice. REST is not dependent on any protocol, but almost every RESTful service uses HTTP as its underlying protocol. It's also very popular in modern API oriented application development.

In this article, I will be discussing 2 different technologies to implement a RESTful microservice and discuss the pros and cons of both approaches. Two technologies which I am going to use;
 1. Apache Camel+SpringBoot
 2. Ballerina

[**SpringBoot**](https://spring.io/projects/spring-boot) is an organized Java framework for building microservices-based on the Spring Dependency Injection framework with minimal code and configuration and allows packaging them into an isolated executable jar giving you production-ready microservices. [**Apache Camel**](https://camel.apache.org/) is a routing engine which provides an implementation for almost all Enterprise Integration Patterns with many different components to facilitate integration with various systems.

[**Ballerina**](https://ballerina.io) is a programming language to create network-distributed applications. It is not a DSL but it’s a full programming language with a rich set of general-purpose features which is optimized for “network distributed applications”

# What we’ll build
To understand how we can build a RESTful microservice, let’s consider a real-world use case of an order management scenario of an online retail application. Following list covers all functionalities in an order management system.
* Create an order
* Retrieve an order
* Update an order
* Delete an order

The following diagram illustrates all the required functionality of the Order Management RESTful microservice that we are going to build.

![order mgt](https://github.com/lakwarus/ballerina-camel-springboot-restful-service/blob/master/images/Order-mgt.png)

NOTE : To reduce length and complecity of this repo/article, cloud native deployment comparison is done at [Cloud Native deployment - Ballerina vs Camel+SpringBoot](https://github.com/lakwarus/ballerina-camel-springboot-cloud-native-deployment) repo/article.

# Apache Camel+SpringBoot
Here we will create a Camel REST microservice using REST DSL, further we will use Camel Servlet to expose the REST API.

## Prerequisites
- JDK 1.8+
- Maven 3.2+
- Your favorite IDE, I have used Spring Tool Suite (STS)

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

When we initialized the project, Spring Boot has created SpringbootCamelRestdslApplication bootstrap class with main() method to run the application. We will now inject Camel Servlet and Camel route to this class.

```java
package com.lakwarus.sample.pojo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootCamelRestdslApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootCamelRestdslApplication.class, args);
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

I have used camel-servlet REST DSL which acts as facade for REST endpoints.

Camel has a way to set CamelServlet registration from Camel version 2.19.0. It is default registered at “/camel” endpoint which you can optionally overwrite. I have overwritten it to /*. Edit src/main/resources/application.properties and add the below property.

```
camel.component.servlet.mapping.context-path=/*
```

To configure REST DSL to use the Servlet component implementation I have used restConfiguration().component(“servlet”). Then I add bindingMode(RestBindingMode.json) tell Camel to format the incoming and outgoing POJOs to JSON format. Binding to/from JSON to be enabled requires a json capabile data format on the classpath. By default Camel use json-jackson as the data format.

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

I have created two resources classes (Order.class and Status.class) which is used to format incoming and outgoing POJOs to JSON format.  

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

The exception handling for Apache camel can be implemented in 2 ways.
* Using Do Try block
* Using OnException block

The OnException applies to all the routes. I have defined OnException block as a separate block from the routes to handle exceptions on malformed JSON requests and any other connection issues. 

To complete our application, let's look at how can add the camel route. Notice that in our REST service we route directly to a Camel endpoint using the to(). This is because the Rest DSL has a short-hand for routing directly to an endpoint using to(). An alternative is to embed a Camel route directly using route()

Here is the full implementation of create order functionality.

```java
	    // Resource that handles the HTTP POST requests that are directed to the path
	    // '/order' to create a new Order.         
	    rest("/ordermgt").consumes("application/json").post("/order").type(Order.class).to("direct:addOrder");

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

We can implement the business logic of each resource depending on our requirements. For simplicity, I used an in-memory map to keep all the order details. (In the real world we should use some Database service to store orders)

For each individual route I have used Do Try blocks to handle exceptions. This approach is similar to the Java try catch block. So the thrown exception will be immediately caught and the message wont keep on retrying. Here I have defined a custom Camel exceptions to handle order validation based exceptions. We have to use multiple Do-Try blocks to handle specific types of exceptions.

Following code block shows implementation of get order functionality.

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

Same way, I have implemented update-order and the delete-order functionalities and full source code of the application can be found [here](https://github.com/lakwarus/ballerina-camel-springboot-restful-microservice/blob/master/springboot-camel-restdsl/src/main/java/com/lakwarus/sample/pojo/SpringbootCamelRestdslApplication.java). 

Note: writting unit tests and comparision will discuss in seperate article.

## Testing
We can run the RESTful microservice that we developed above, in our local environment. 

Open SpringbootCamelRestdslApiApplication class and Run as a Java Application or SpringBoot Application. Check the console tab for application build status. 

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
$ curl -v "http://localhost:8080/ordermgt/order/100500"

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
$ curl -X PUT -d '{ "ID": "100500", "Name": "XYZ", "Updated Description": "Sample order."}' http://localhost:8080/ordermgt/order/100500 -H "Content-Type:application/json" -v

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
$ curl -X DELETE "http://localhost:8080/ordermgt/order/100500" -v

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

# Ballerina

## Prerequisites

- Ballerina Distribution
- A Text Editor or an IDE. I have used VSCode with Ballerina VSCode plugin

## Setting up the project

Ballerina project is a directory that atomically manages a collection of modules and programs. We can create a project from any folder by using the following command.

```bash
$> ballerina new restful-service
Created new ballerina project at restful-service

Next:
    Use `ballerina create` to create a ballerina module inside the project.
$> cd restful-service
$> ballerina create order_mgt
Created new ballerina module at 'src/order_mgt'
```

I have created `restful-service` project inside `ballerina-restful-service` folder. Then I create `order_mgt` module and create `order_service.bal` source file of my order management application.

## Implementation

Unlike Java, Ballerina source itself can import modules. Since I am going to create an HTTP Restful service I have imported the ballerina/http module. Endpoints and Services are first-class constructs in Ballerina. I have created `httpListener` endpoint and set listener port as 8080. Then I have created  `orderMgt` service and set it to listen on `httpListener` endpoint. 

```ballerina
import ballerina/http;

listener http:Listener httpListener = new(8080);

@http:ServiceConfig { basePath: "/ordermgt" }
service orderMgt on httpListener {

```
Ballerina comes with built-in annotation support. Here I have used `@http:ServiceConfig` annotation to set based path of our rest service.

A service can have any number of resource functions. We can implement the business logic of each resource function depending on our requirements. For simplicity, I have used an in-memory map to keep all the order details.

Following code block shows implementation of `addOrder` resource function. Here, you will see how certain HTTP status codes and headers are manipulated whenever required in addition to the order processing logic. I have used `@http:ResourceConfig` to set the resource path and the HTTP method.

```ballerina
    // Resource that handles the HTTP POST requests that are directed to the path
    // '/order' to create a new Order.
    @http:ResourceConfig {
        methods: ["POST"],
        path: "/order"
    }
    resource function addOrder(http:Caller caller, http:Request req) {
        http:Response response = new;
        var orderReq = req.getJsonPayload();
        if (orderReq is json) {
            
            json|error orderIdJson = orderReq.id;
            if (orderIdJson is error) {

                // Create response message.
                json payload = { status: "OrderId is Empty!", orderId: "null" };
                response.statusCode = 400;
                response.setJsonPayload(payload);
            } else {

                string orderId = orderIdJson.toString();

                // Find the duplicate orders
                json existingOrder = ordersMap[orderId];

                if (existingOrder == null) {
                    ordersMap[orderId] = orderReq;

                    // Create response message.
                    json payload = { status: "Order Created.", orderId: orderId };
                    response.setJsonPayload(payload);

                    // Set 201 Created status code in the response message.
                    response.statusCode = 201;
                    // Set 'Location' header in the response message.
                    // This can be used by the client to locate the newly added order.
                    response.setHeader("Location", "http://localhost:8080/ordermgt/order/" +
                            orderId);
                } else {
                    response.statusCode = 500;
                    json payload = { status: "Duplicate Order", orderId: orderId };
                    response.setJsonPayload(payload);
                }
            }
        } else {
            response.statusCode = 400;
            json payload = { status: "Invalid JSON received!", orderId: "null" };
            response.setJsonPayload(payload);
        }
        // Send response to the client.
        var result = caller->respond(response);
        if (result is error) {
            log:printError("Error sending response", err = result);
        }
    }	
```

Ballerina has a single type named json that can represent any JSON value. Thus, json is a built-in union type in Ballerina that can contain values of types nil (as the null value), boolean, int, float, decimal, string, json\[\] and map<json>. Because JSON is built-in type, it does not require any additional module to import.
	
In Ballerina, errors can be returned or can cause abrupt completion via panic. In above code block, I returned errors and logged them.

TODO : write TEST

[Here](https://github.com/lakwarus/ballerina-camel-springboot-restful-microservice/blob/master/ballerina-restful-service/restful-service/src/order_mgt/order_service.bal), you can find the full source code of the order management service

Note: writting unit tests and comparision will discuss in seperate article.

## Testing

We can run the RESTful microservice that we developed above, in our local environment. Open the terminal execute the following command.

```bash
$> ballerina build
Compiling source
    lakmal/order_mgt:0.1.0
Created target/balo/order_mgt-2019r3-any-0.1.0.balo

Running tests
    lakmal/order_mgt:0.1.0
I'm the before suite function!
I'm the before function!
I'm in test function!
I'm the after function!
I'm the after suite function!
	1 passing
	0 failing
	0 skipped

Generating executables
target/bin/order_mgt-executable.jar
$> ballerina run target/bin/order_mgt-executable.jar
[ballerina/http] started HTTP/WS listener 0.0.0.0:8080
```
  
To test the functionality of the orderMgt RESTFul service, send HTTP requests for each order management operation. Following are sample cURL commands that we can use to test each operation of the order management service.

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
< HTTP/1.1 400 Bad Request
< content-type: application/json
< content-length: 48
< server: ballerina/0.991.0
< date: Thu, 1 Aug 2019 11:47:00 -0700
< 
* Connection #0 to host localhost left intact
{"status":"OrderId is Empty!", "orderId":"null"}
```

#### Retrieve Order

```bash
$ curl -v "http://localhost:8080/ordermgt/order/100500"

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /ordermgt/order/100500 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< content-type: application/json
< content-length: 55
< server: ballerina/0.991.0
< date: Thu, 1 Aug 2019 11:47:49 -0700
< 
* Connection #0 to host localhost left intact
{"status":"Order cannot be found!", "orderId":"100500"}
```

#### Update Order

```bash
$ curl -X PUT -d '{ "ID": "100500", "Name": "XYZ", "Updated Description": "Sample order."}' http://localhost:8080/ordermgt/order/100500 -H "Content-Type:application/json" -v

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> PUT /ordermgt/order/100500 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 72
> 
* upload completely sent off: 72 out of 72 bytes
< HTTP/1.1 200 OK
< content-type: application/json
< content-length: 55
< server: ballerina/0.991.0
< date: Thu, 1 Aug 2019 11:50:53 -0700
< 
* Connection #0 to host localhost left intact
{"status":"Order cannot be found!", "orderId":"100500"}
```

#### Delete Order

```bash
$ curl -X DELETE "http://localhost:8080/ordermgt/order/100500" -v

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> DELETE /ordermgt/order/100500 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< content-type: application/json
< content-length: 55
< server: ballerina/0.991.0
< date: Thu, 1 Aug 2019 11:51:32 -0700
< 
* Connection #0 to host localhost left intact
{"status":"Order cannot be found!", "orderId":"100500"}
```

## Comparison

Both Camel+SpringBoot and Ballerina provide a very rich user experience to create Restful microservices. Both option has its own pros and cons. In this section I am trying to compare the experience I had with both options with this sample.

#### Initializing the project:
Both option has very simple flow to initialize a project.
Camel+SpringBoot has its own tool (STS - eclipse based) and Ballerina provide plugin to existing tools.

#### Dependency management:
Camel+SpringBoot required dependency management by updating pom.xml and then import dependant packages in source. Ballerina dependency management is very simple compared to Camel+SpringBoot. Just importing dependent modules in the source will works.

#### Initialize a Rest service:
Camel+SpringBoot has multiple ways to initialize and implement a Rest service. Its give more flexibility to the developer to choose suitable solution. While this is a great flexibility, but its come with a complexity. I had to spend a significant amount of time to choose the best option suited for my application. But I could not come to a conclusion and then finally just use one option in my sample. 
Service is a first class construct in Ballerina and its straightforward to initialize a Rest service. 

#### Configure Rest API properties:
Both Camel+SpringBoot and Ballerina provide a very simple way to set/define Rest API properties like endpoints, paths, methods, etc.

#### JSON binding:
Again, Camel+SpringBoot has multiple ways to bind JSON and which confused me for a while. Finally I used Beans option which is simplified the rest of my application.
JSON data type is a built-in type in Ballerina and handling JSON is very simple and intuitive.

#### Readability and structure of the code: 
Ballerina has opinionated way to structure our code and it give consistent and provide very good readability. (eg. resource function has Rest API properties and business logic in a single structure and provide better readability)
Camel+SpringBoot has more flexible on structure our code but some time its lead to less readability. 

#### Error Handling: 
Comparing both options, Ballerina error handling is more comprehensive and intuitive. In Ballerina all network related functionality returns errors and guide developers to explicitly handle errors. Which reduce many developer mistakes and runtime errors. 
Camel+SpringBoot also has flexibility on error handling but guidedency is weak. This might let developer mistakes and finally it can be lead to runtime errors.  

#### Community:
Spring+Camel has a good amount of community samples compared to Ballerina.   


# Summary

* This article mainly compared Apache Camel+SpringBoot with Ballerina context of creating restfull microservices. 
Comparision was carried out by implementing real world sample by using borth technologies.

* In the context of restfull microservices, Ballerina has advantage over Apache Camel+SpringBoot due to Ballerina language design by ground up for network distributed application.
 

# References
- http://www.javaoutofbounds.com/apache-camel-springboot-rest-api-example/
- https://www.javainuse.com/camel/camelException
- https://camel.apache.org/rest-dsl.html
- https://ballerina.io/learn/by-guide/restful-service


