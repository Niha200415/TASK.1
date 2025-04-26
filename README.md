import akka.actor.ActorSystem //provides the actor system for handling concurrency.

import akka.http.scaladsl.Http //provides the HTTP server and client functionality.

import akka.http.scaladsl.model._ //This imports various models related to HTTP, such as requests and responses.

import akka.http.scaladsl.server.Directives._ //This imports directives that are used to define routes for the HTTP server.

import akka.stream.ActorMaterializer //This imports the ActorMaterializer, which is required for materializing streams.


import spray.json._ //This imports the Spray JSON library, which is used for JSON serialization and deserialization.


import org.apache.kafka.clients.producer.{KafkaProducer, ProducerRecord} //This imports classes needed to create a Kafka producer, which                                                                                                                             is used for sending messages to Kafka topics.

import com.rabbitmq.client.ConnectionFactory //This imports the ConnectionFactory class from RabbitMQ, which is used to create         
                                                    connections to RabbitMQ servers.

case class ContainerRoutingRequest(containers: List[Int], routes: List[List[Int]]) //This defines a case class for the incoming request,                                                                                    which contains a list of container IDs and their routes.

case class ContainerRoutingResponse(optimalRoute: List[Int]) //This defines a case class for the response, which contains the optimal                                                                     route as a list of integers.


object ContainerRoutingAPI { //This defines a singleton object that will hold the API functionality.
 
  implicit val system = ActorSystem("container-routing-api") // This creates an implicit ActorSystem named "container-routing-api",                                                                       which is used throughout the application.

  implicit val materializer = ActorMaterializer() //This creates an implicit ActorMaterializer that is necessary for handling streams.

  implicit val executionContext = system.dispatcher // This sets the execution context to the dispatcher of the ActorSystem, which is                                                             used for executing futures.

  // Kafka producer
  val props = new java.util.Properties() //This creates a new Properties object to configure the Kafka producer.
  
  props.put("bootstrap.servers", "localhost:9092") //This sets the address of the Kafka broker.


  props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer") //This specifies the serializer for the keys of                                                                                             the messages sent to Kafka.
 
  props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer") //This specifies the serializer for the values                                                                                               of the messages sent to Kafka.
 
  val kafkaProducer = new KafkaProducer[String, String](props) //This creates a new Kafka producer with the specified properties.

  // RabbitMQ connection
  
  val factory = new ConnectionFactory() //This sets the RabbitMQ server host to localhost.
  
  factory.setHost("localhost") //This sets the RabbitMQ server host to localhost.
 
  val rabbitMQConnection = factory.newConnection() //This establishes a new connection to the RabbitMQ server.
 
  val channel = rabbitMQConnection.createChannel() //This  creates a new channel for communication with RabbitMQ.


  val route = path("optimize") { // This defines a route for the API that listens for POST requests at the path "/optimize".
    
    post { //This specifies that the following directives apply to POST requests.
     
      entity(as[String]) { request => //This extracts the request body as a string.
        
        val containerRoutingRequest = request.parseJson.convertTo[ContainerRoutingRequest] //This parses the JSON request body into a                                                                                                     ContainerRoutingRequest object.
        val optimalRoute = optimize(containerRoutingRequest.containers, containerRoutingRequest.routes) //This calls a method optimize 
        (not shown in the snippet) to calculate the optimal route based on the containers and routes provided in the request.
        
        val response = ContainerRoutingResponse(optimalRoute)

        // Send response to Kafka topic
        kafkaProducer.send(new ProducerRecord[String, String]("container-routing-response", response.toJson.compactPrint))

        // Send response to RabbitMQ queue
        channel.basicPublish("", "container-routing-response", null, response.toJson.compactPrint.getBytes)

        complete(response.toJson.compactPrint)
      }
    }
  }

  def optimize(containers: List[Int], routes: List[List[Int]]): List[Int] = {
   
    // Implement the mathematical formulas for container-routing optimization here
   
    // For example:
  
    val numContainers = containers.size
  
    val numRoutes = routes.size
   
    val costMatrix = Array.ofDim[Int](numContainers, numRoutes)
    // ...
    // Calculate the optimal route using the cost matrix and other variables
    // ...
    List(1, 2, 3) // Return a sample optimal route
  }

  def main(args: Array[String]) {
    
    val bindingFuture = Http().bindAndHandle(route, "localhost", 8080)
   
    println(s"Server online at http://localhost:8080/\nPress RETURN to stop...")
   
    scala.io.StdIn.readLine()
   
    bindingFuture
      
      .flatMap(_.unbind())
      
      .onComplete(_ => system.terminate())
  }
}


build.sbt

name := "container-routing-api"

version := "0.1"

scalaVersion := "2.13.8"

libraryDependencies ++= Seq(
 
  "com.typesafe.akka" %% "akka-http" % "10.2.9",
  
  "com.typesafe.akka" %% "akka-stream" % "2.6.19",
 
  "io.spray" %% "spray-json" % "1.3.6",
  
  "org.apache.kafka" %% "kafka" % "3.1.0",
 
  "com.rabbitmq" % "amqp-client" % "5.14.2"
)


README.md

 Overview
 This API provides a container routing optimization solution using Akka HTTP, Kafka, and RabbitMQ.

Request/Response Example

Request:

json
{
  
  "containers": [1, 2, 3],
  
  "routes": [[1, 2], [2, 3], [1, 3]]
}


Response:

{
  "optimalRoute": [1, 2, 3]
}
