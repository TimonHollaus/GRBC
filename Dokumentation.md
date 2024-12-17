DEZSYS_GK72_ELECTION_GRPC

This lesson introduces Remote Procedure Call (RPC) technology.

Introduction
This exercise is intended to demonstrate the functionality and implementation of Remote Procedure Call (RPC) technology using the open-source gRPC framework (https://grpc.io). It shows how this framework can be used to develop a middleware system that connects multiple services developed in different programming languages.

Document all individual implementation steps and any problems that arise in a log (Markdown). Create a GitHub repository for this project and include the link in the comments.

Assessment Requirements mostly fulfilled
Answer the questions below about the gRPC framework:

Use one of the tutorials listed under "Links & Further Resources" to create a simple HelloWorld application using the gRPC framework.
First, create the Proto file where you define your gRPC service and its data structures.
Implement a gRPC server and a gRPC client in the programming language of your choice.
Document each individual development step in your protocol and describe the most important code snippets in a few sentences. Additionally, the output of the application and any problems that occur should be documented in the submission.
Requirements fully fulfilled
Customize the service so that a simple ElectionData record can be transferred. Document which parts of the program need to be adapted.

Documentation
Create an IntelliJ project with Gradle and import the gRPC zip in Project-Structure → Libraries.

Then, edit the build.gradle.kts file with the dependencies:

kotlin
Code kopieren
implementation("io.grpc:grpc-netty:1.68.1")
implementation("io.grpc:grpc-protobuf:1.68.1")
implementation("io.grpc:grpc-stub:1.68.1")
implementation("io.grpc:grpc-api:1.68.1")
compileOnly("javax.annotation:javax.annotation-api:1.3.2")
with the plugin:

kotlin
Code kopieren
id("com.google.protobuf") version "0.9.4"
and with sourceSets:

kotlin
Code kopieren
import com.google.protobuf.gradle.id

plugins {
id("java")
id("com.google.protobuf") version "0.9.4"
}

group = "org.example"
version = "1.0-SNAPSHOT"

repositories {
mavenCentral()
}

dependencies {
testImplementation(platform("org.junit:junit-bom:5.9.1"))
testImplementation("org.junit.jupiter:junit-jupiter")

    implementation("io.grpc:grpc-netty:1.68.1")
    implementation("io.grpc:grpc-protobuf:1.68.1")
    implementation("io.grpc:grpc-stub:1.68.1")
    implementation("io.grpc:grpc-api:1.68.1")
    compileOnly("javax.annotation:javax.annotation-api:1.3.2")
}
sourceSets {
main {
java {
srcDir("build/generated/source/proto/main/java")
srcDir("build/generated/source/proto/main/grpc")
}
}
}

tasks.test {
useJUnitPlatform()
}
Then, click “Load Gradle Changes” to ensure the project is set up correctly.

Next, add the protobuf configuration to the Gradle file:

kotlin
Code kopieren
protobuf {
protoc {
artifact = "com.google.protobuf:protoc:3.25.5"
}
plugins {
id("grpc") {
artifact = "io.grpc:protoc-gen-grpc-java:1.68.1"
}
}
generateProtoTasks {
all().forEach { task ->
task.plugins {
id("grpc")
}
}
}
}
If it doesn't work initially, make sure to import the protobuf.gradle.id library:

kotlin
Code kopieren
import com.google.protobuf.gradle.id
Now, create a new directory called proto in src and add a hello.proto file where you define the HelloWorldService and the messaging services HelloRequest and HelloResponse:

proto
Code kopieren
syntax = "proto3";

option java_package = "org.example";
option java_outer_classname = "Hello";

service HelloWorldService {
rpc hello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
string firstname = 1;
string lastname = 2;
}

message HelloResponse {
string text = 1;
}
Afterward, reload Gradle changes again to continue the setup.

Create the following 3 classes in the main/java/org/example directory:

HelloWorldClient.java
This class connects to the server using ManagedChannel, calls the hello method, and sends a HelloRequest with the first and last name. It then receives a HelloResponse containing the greeting text and prints it to the console. The connection is closed after the response is received.
java
Code kopieren
package org.example;

import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;

public class HelloWorldClient {

    public static void main(String[] args) {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 50051)
                .usePlaintext()
                .build();

        HelloWorldServiceGrpc.HelloWorldServiceBlockingStub stub = HelloWorldServiceGrpc.newBlockingStub(channel);

        Hello.HelloResponse helloResponse = stub.hello(Hello.HelloRequest.newBuilder()
                .setFirstname("Nico")
                .setLastname("Furtner")
                .build());
        System.out.println(helloResponse.getText());
        channel.shutdown();
    }
}
ServerClass.java
This class implements the HelloWorldService by defining the hello method. The server processes the request and generates a message using the first and last name provided in the HelloRequest.
java
Code kopieren
package org.example;

import io.grpc.Server;
import io.grpc.ServerBuilder;

import java.io.IOException;

public class ServerClass {

    private static final int PORT = 50051;
    private Server server;

    public void start() throws IOException {
        server = ServerBuilder.forPort(PORT)
                .addService(new HelloWorldService())
                .build()
                .start();

        System.out.println("Server started, listening on port " + PORT);
    }

    public void blockUntilShutdown() throws InterruptedException {
        if (server == null) {
            return;
        }
        server.awaitTermination();
    }

    public static void main(String[] args) throws InterruptedException, IOException {
        ServerClass server = new ServerClass();
        System.out.println("Server is running with both HelloWorld and Election services!");
        server.start();
        server.blockUntilShutdown();
    }
}
HelloWorldService.java
This class contains the hello method that handles requests from the client and returns a response.
java
Code kopieren
package org.example;

import io.grpc.stub.StreamObserver;

public class HelloWorldService extends HelloWorldServiceGrpc.HelloWorldServiceImplBase {

    @Override
    public void hello(Hello.HelloRequest request, StreamObserver<Hello.HelloResponse> responseObserver) {
        System.out.println("Handling hello endpoint: " + request.toString());

        String text = "Hello World, " + request.getFirstname() + " " + request.getLastname();
        Hello.HelloResponse response = Hello.HelloResponse.newBuilder().setText(text).build();

        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
Before starting the server, it's important to build the Gradle project.

Start the server and the client, and you should see the greeting message displayed.

We use port 50051 because it is specified in the documentation and is the only port supported for gRPC.

Make sure to create the files and directories in the correct location. I initially faced an issue because I created the proto directory in main, which didn’t work. Once I created it in src, everything worked.

Helpful source: Intuting Medium Guide