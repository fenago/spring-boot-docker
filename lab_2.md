
Lab: Deploying Multiple Spring Boot Microservices to Docker using Docker Networking
===================================================================================


In a previous docker lab, we
saw how to deploy a single Spring Boot Microservice to Docker Container.
But suppose the requirement is such that we may need to deploy multiple
microservices to Docker Container. For example we have the following
microservices that we have defined in previous tutorial,
As the name suggests employee-producer will be exposing REST APIs which
will be consumed by the employee-consumer.
The way Docker has been designed such that a Docker Container should
have only a single service running. Again we can have multiple services
running in docker using some workarounds but this will not be a good
design. So we will be deploying the two microservices employee-producer
and employee-consumer to two different containers and then have them
interact with each other.

![](./images/docker-5_2.jpg "Docker Networking Tutorial")



In order to achieve this will have to make use of the **docker networking commands.**

**Lets Begin**

**Note:** Make sure to stop and delete all running container(s) first.

`docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)`

Lets begin with the implementation part.

-   **Employee Producer -**

Now open the terminal and go to the Spring Boot employee-producer
project folder.
Next we will build an image with the name producer.

```
cd ~/spring-boot-docker/employee-producer

docker image build -t employee-producer .
```

![](./images/docker-image-build.jpg)
Next we will run the above image as a container named producer. Also we
will be publishing the docker port 8080 to centos port 8080.

```
docker container run --name producer -p 8080:8080 -d employee-producer
```

![](./images/docker-employee-build.jpg)

![](./images/docker-container-running2.jpg)



So our employee container has started. We can test this by going to
localhost:8080/employee, we will see that our application is deployed
successfully.

![](./images/docker-boot-container-tomcat.jpg)

**Employee Consumer -**


![](./images/docker-boot-employee.jpg)
We have created and started a container named **producer** where we have
deployed the employee-producer service.
So the only change we will be making is while consuming the employee
producer service we will be using this container named producer instead
of localhost:8080.
So in the ConsumerControllerClient class we will be having the base url
as **http://producer:8080/employee** instead of
http://localhost:8080/employee.

```
package com.javainuse.controllers;

import java.io.IOException;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.RestClientException;
import org.springframework.web.client.RestTemplate;

public class ConsumerControllerClient {

    public void getEmployee() throws RestClientException, IOException {

        String baseUrl = "http://producer:8080/employee";
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> response=null;
        try{
        response=restTemplate.exchange(baseUrl,
                HttpMethod.GET, getHeaders(),String.class);
        }catch (Exception ex)
        {
            System.out.println(ex);
        }
        System.out.println(response.getBody());
    }

    private static HttpEntity<?> getHeaders() throws IOException {
        HttpHeaders headers = new HttpHeaders();
        headers.set("Accept", MediaType.APPLICATION_JSON_VALUE);
        return new HttpEntity<>(headers);
    }
}
```

The docker file is as follows-

```
FROM openjdk:8

RUN apt-get update
RUN apt-get install -y maven
WORKDIR /demo
COPY . /demo
RUN mvn clean install
CMD ["java","-jar","/demo/target/employee-consumer-0.0.1-SNAPSHOT.jar"]
```

![](./images/docker-dockerhub.jpg)



Open the terminal and go to the Spring Boot employee consumer project
folder.
Next we will build an image with the name consumer.

```
cd ~/spring-boot-docker/employee-consumer

docker image build -t employee-consumer .
```

![](./images/docker-consumer-build.jpg)

Next we will run the above image as a container named consumer.

```
docker container run --name consumer -d employee-consumer
```

![](./images/docker-consumer-run.jpg)

Next check the logs using

```
docker container logs consumer
```

![](./images/docker-consumer-logs.jpg)

Here we can see that the container named consumer is not able to
communicate with the container named producer.

![](./images/docker-consumer-not-communicate.jpg)

So we are getting a null pointer exception.


**Inter Docker Container Communication Using Docker Networking**

We will be using Docker Networking to allow multiple containers to
interact with each other.

![](./images/docker-networking-tutorial.jpg)

We will need to create our own network and add both the
employee-producer and employee-consumer services to it. We will stop and
remove the existing docker containers named consumer and producer.

**Note:** Make sure to stop and delete running container(s) first.


![](./images/docker-consumer-remove.jpg)

```
docker container stop producer

docker container rm producer
```

![](./images/docker-producer-remove.jpg)

```
docker container stop consumer

docker container rm consumer
```

Lets first check the available networks

```
docker network ls
```

![](./images/docker-network-list.jpg)
Next we will create our own network will be of type bridge

```
docker network create consumer-producer
```

![](./images/docker-create-network.jpg)
Lets start the employee producer container on the newly created network.

```
docker container run --network consumer-producer --name producer -p 8080:8080 -d employee-producer
```

![](./images/docker-create-network-producer.jpg)

Lets start the employee consumer+ container on the newly created
network.

```
docker container run --network consumer-producer --name consumer -d employee-consumer
```

![](./images/docker-create-network-consumer.jpg)



Lets check the consumer container logs-

```
docker container logs consumer -f
```

![](./images/docker-network-logs.jpg)










