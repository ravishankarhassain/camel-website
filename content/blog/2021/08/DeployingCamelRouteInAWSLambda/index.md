---
title: "Deploying a Camel Route in AWS Lambda using Quarkus"
date: 2021-08-26
authors: [ravishankarhassain]
categories: ["AWS","Camel Quarkus"]
preview: "A new demonstration explaining how to develop a camel route using Quarkus (in both JVM & Native mode) and steps to deploy it in AWS Lamda"
---

Do you fancy running camel route as functions in AWS Lambda. Well I did a small Proof Of Concept to test this and the results were interesting.
Thanks to the Quarkus and Camel-Quarkus communities for their efforts to make this technically possible.


You can find the working sample in the Camel Quarkus Examples [github repo](https://github.com/apache/camel-quarkus-examples/tree/camel-quarkus-main/aws-lambda)

#Deploying a Camel Route in AWS Lambda : A Camel Quarkus example

This project uses the following framework 

 * **Quarkus** - *The Supersonic Subatomic Java Framework for building Cloud Native Applications*
 * **Apache Camel** - *The Swiss Army Knife of Enterprise Application Integration for integrating heterogeneous systems*
 * **AWS Lambda** - *Event-driven, serverless computing platform provided by Amazon as a part of Amazon Web Services*

If you want to learn more about 

* *Quarkus - please visit its website: https://quarkus.io/*
* *Apache Camel - please visit its website: https://camel.apache.org/*
* *AWS Lambda - please visit its website: https://aws.amazon.com/lambda/*

## Provided Code

### Quarkus Camel Amazon Lambda Integration example

This example contains a sample Greeter service build using Quarkus & Camel framework implementing Dependency Injection design principle which can be deployed as functions into the Amazon Lambda.

> :warning: **INCOMPATIBLE WITH DEV MODE**: Amazon Lambda Binding is not compatible with dev mode yet!


#### Quarkus Extensions / Dependencies Used

1. **[Apache Camel](https://quarkus.io/guides/camel)**
    * [camel-quarkus-core](https://camel.apache.org/camel-quarkus/latest/reference/extensions/core.html)
    * [camel-quarkus-direct](https://camel.apache.org/camel-quarkus/latest/reference/extensions/direct.html)
    * [camel-quarkus-log](https://camel.apache.org/camel-quarkus/latest/reference/extensions/log.html)
    * [camel-quarkus-bean](https://camel.apache.org/camel-quarkus/latest/reference/extensions/bean.html)

2. **Amazon Lambda**
    * [quarkus-amazon-lambda](https://quarkus.io/guides/amazon-lambda)
    
3. **[Context Dependency Injection](https://quarkus.io/guides/cdi)**
    * [quarkus-arc](https://quarkus.io/guides/cdi-reference)
    
4. **[Testing](https://quarkus.io/guides/getting-started-testing)**
    * quarkus-junit5
    * [quarkus-junit5-mockito](https://quarkus.io/blog/mocking/)
    * quarkus-test-amazon-lambda

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```shell script
./mvnw compile quarkus:dev
```

> **_NOTE:_**  Quarkus now ships with a Dev UI, which is available in dev mode only at http://localhost:8080/q/dev/.

## Building and Packaging the Java code as Quarkus JVM application

The application can be packaged using:
```shell script
./mvnw clean package
```
This will compile and package your code. 

It produces the `quarkus-run.jar` file in the `target/quarkus-app/` directory.
Be aware that it’s not an _uber-jar_ as the dependencies are copied into the `target/quarkus-app/lib/` directory.

If you want to build an _uber-jar_, execute the following command:
```shell script
./mvnw package -Dquarkus.package.type=uber-jar
```

The application is now runnable using `java -jar target/quarkus-app/quarkus-run.jar`.

It also generates a zip file target/function.zip. This zip file contains your java code along with the dependencies.

##<a name="native"> </a>Building and Packaging the Java code as Quarkus Native executable

If you want a lower memory footprint and faster initialization times for your lambda, you can compile your Java code to a native executable. Just make sure to rebuild your project with the -Pnative switch.
> :warning: **Building Native Executables will take much longer time and depends on the underlying system**
```shell script
./mvnw package -Pnative
```
> :information_source: If you are building on a non-Linux system Or, if you don't have GraalVM installed, you can run the native executable build using docker container. You need to pass in a property instructing quarkus to use a docker build as Amazon Lambda requires linux binaries. 
>You can do this by passing this property to your Maven build: `-Dquarkus.native.container-build=true`. However, This requires to have docker installed locally.

```shell script
./mvnw package -Pnative -Dquarkus.native.container-build=true
```

Either of these commands will compile and create a native executable image. 
You can then execute your native executable with: `./target/code-with-quarkus-1.0.0-SNAPSHOT-runner`

It also generates a zip file target/function.zip. This zip file contains your native executable image renamed to bootstrap. This is a requirement of the AWS Lambda Custom (Provided) Runtime.

If you want to learn more about building native executables, please consult https://quarkus.io/guides/maven-tooling.html.

## Extra Build Generated Files
 **_NOTE:_** After you run the build, there are a few extra files generated by the quarkus-amazon-lambda extension. These files are in the build directory: `target/`

* function.zip - lambda deployment file

* manage.sh - wrapper around aws lambda cli calls

* bootstrap-example.sh - example bootstrap script for native deployments

* sam.jvm.yaml - (optional) for use with sam cli and local testing

* sam.native.yaml - (optional) for use with sam cli and native local testing

> :information_source: [Please click here to know more on how to use these scripts for automated deployment](https://quarkus.io/guides/amazon-lambda#extra-build-generated-files)

## Deploying the Quarkus JVM application to AWS Lambda via AWS Web Console

1. Go to AWS Web console and search for Lambda Service
 
2. Click Create Function and select Author From Scratch 

3. Give the name for your function which should be unique

4. Select Java 11 (Corretto) as Runtime 

5. Under Permission feel free to create / use existing role to give the required permission for your lambda function

6. Once the function is created click the function name to upload the generated function.zip file and configure it. 

7. Scroll down and select the Code tab. Click the upload from dropdown on right hand side of the screen and select .zip or .jar file 

8. Click upload and browse to the path where the generated zip file is created `target/function.zip` and select the function.zip file and click save

9. Under the Code tab scroll down to the Runtime settings and click edit 

10. For the Handler details please provide the Quarkus Handler 

```shell script
io.quarkus.amazon.lambda.runtime.QuarkusStreamHandler::handleRequest
```

### Testing the AWS Lamda Function

1. Now select the Test tab for executing a quick test. Copy paste the below json payload and hit Test 

    ```shell script
    {
     "name": "Ravishankar"
    }
    ```
2. If everything goes fine you should get the below response along with the lambda execution logs & stats

    ```shell script
    Hello Ravishankar ! How are you? from GreetService
    ```
## Deploying the Quarkus Native executable to AWS Lambda via AWS Web Console

**_NOTE:_** [Please ensure that you have built your Java code as Quarkus Native executable](#native)

1. Go to AWS Web console and search for Lambda Service
 
2. Click Create Function and select Author From Scratch 

3. Give the name for your function which should be unique

4. For Runtime please scroll down and Select Provide your own bootstrap on Amazon Linux 2 under Custom Runtime 

5. Under Permission feel free to create / use existing role to give the required permission for your lambda function

6. Once the function is created click the function name to upload the generated function.zip file and configure it. 

7. Scroll down and select the Code tab. Click the upload from dropdown on right hand side of the screen and select .zip or .jar file 

8. Click upload and browse to the path where the generated a zip file is created `target/function.zip` and select the function.zip file and click save

9. Under the Code tab scroll down to the Runtime settings and click edit 

10. For the Handler details please provide the below Handler 

    ```shell script
        not.used.in.provided.runtime
    ```

11. Then Select the Configuration tab and click Environment Variables 

12. For Key enter `DISABLE_SIGNAL_HANDLERS`	& for Value enter `true`

### Testing the AWS Lamda Function

1. Now select the Test tab for executing a quick test. Copy paste the below json payload and hit Test 

    ```shell script
    {
     "name": "Ravishankar"
    }
    ```
2. If everything goes fine you should get the below response along with the lambda execution logs & stats

    ```shell script
    Hello Ravishankar ! How are you? from GreetService
    ```

## JVM vs Native : Results based on lambda execution logs & stats
![JVM vs Native Results](results.png?raw=true "JVM vs Native Results")

The above table is populated based on the execution stats provided in the AWS Lamda Logs 

Based on the data captured sharing few comments:

* It's interesting to see that native uses less memory than JVM which might reduce the cost in running AWS Lambda

* The execution time seems faster in native mode from 2nd to 5th invocations, it would be interesting to see if this trend continues on subsequent calls as the JIT compiler might kick in.

* It's clear that the init duration is by magnitude faster in native mode. However, I am curious to see that for the first call the total billed duration includes the Init duration in native mode where as in JVM mode, Init duration is ignored in the total billed duration. Not sure, If this is a billing issue in AWS or a feature provided by AWS for executing Java runtime in Lambda ?
