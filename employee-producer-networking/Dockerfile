FROM openjdk:8

RUN apt-get update
RUN apt-get install -y maven
WORKDIR /demo
COPY . /demo
RUN mvn clean install
CMD ["java","-jar","/demo/target/employee-producer-0.0.1-SNAPSHOT.jar"]
