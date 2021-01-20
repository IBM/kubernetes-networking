
## How the HelloWorld App was Created

Do not 

1. The following step is only explaining how the HelloWorld application was created. The Kubernetes resources will actually pull an existing image from Docker Hub.

2. Create the Spring Boot scaffolding,
```
$ spring init --dependencies=web,data-rest,thymeleaf helloworld-api
$ cd helloworld-api
$ mvn clean install
$ mvn test
$ mvn spring-boot:run
```

1. Create the APIController,

```
$ echo 'package com.example.helloworldapi;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;

@RestController
public class APIController {

  @Autowired
  private MessageRepository repository;
    
  @GetMapping("/api")
  public String index() {
    return "Welcome to Spring Boot App";
  }

  @RequestMapping(value = "/api/hello", method = RequestMethod.GET,
    produces = MediaType.APPLICATION_JSON_VALUE)
  @ResponseBody
  public String hello(@RequestParam String name) {
    String message = "Hello "+ name;
    String responseJson = "{ \"message\" : \""+ message + "\" }";
    repository.save(new Message(name, message));
    for (Message msg : repository.findAll()) {
        System.out.println(msg);
    }
    return responseJson;
  }

  @RequestMapping(value = "/api/messages", method = RequestMethod.GET,
    produces = MediaType.APPLICATION_JSON_VALUE)
  @ResponseBody
  public ResponseEntity<List<Message>> getMessages() {
    List<Message> messages = (List<Message>) repository.findAll();
    return new ResponseEntity<List<Message>>(messages, HttpStatus.OK);
  }
}
' > src/main/java/com/example/helloworldapi/APIController.java
```

1. Create the Message class,

```
$ echo 'package com.example.helloworldapi;

import org.springframework.data.annotation.Id;

public class Message {

  @Id
  private String id;
  
  private String sender;
  private String message;

  public Message(String sender, String message) {
    this.sender = sender;
    this.message = message;
  }

  public String getId() {
    return id;
  }
  public String getSender() {
    return sender;
  }
  public String getMessage() {
    return message;
  }

  @Override
  public String toString() {
    return "Message [id=" + id + ", sender=" + sender + ", message=" + message + "]";
  }
}' > src/main/java/com/example/helloworldapi/Message.java
```

1. Create the MessageRepository class,

```
echo 'package com.example.helloworldapi;

import java.util.List;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface MessageRepository extends MongoRepository<Message, String> {

	public List<Message> findBySender(String sender);

}' > src/main/java/com/example/helloworldapi/MessageRepository.java
```

1. Add a new file ‘~/src/main/resources/application.properties’, whose configuration should match those when the MongoDB server was deployed,

```
echo 'pring.data.mongodb.username=user1
spring.data.mongodb.password=passw0rd
spring.data.mongodb.database=mydb
spring.data.mongodb.port=27017
spring.data.mongodb.host=mongodb.default' > src/main/resources/application.properties
```

1. Add the following dependency to the Maven build file in `pom.xml`,

    ```
    <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
    </dependencies>
    ```


## Test Java App on localhost:

1. Create local Mongodb,
```
docker run --name mongo -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=passw0rd  mongo
```

1. Get IP address on mac osx,
```
$ ifconfig en0 | grep inet
192.168.86.45
```

1. Configure application.properties and change the host to your local IP Address to test the app on localhost. On the localhost we did not create a separate user, but you can use the admin user.
```
spring.data.mongodb.username=user1
spring.data.mongodb.password=passw0rd
spring.data.mongodb.database=mydb
spring.data.mongodb.port=27017
spring.data.mongodb.host=192.168.86.45
```


1. Run the application
```
$ mvn clean install
$ mvn spring-boot:run
```

1. test
```
$ curl -X GET 'http://192.168.86.45:8080/api/hello?name=one'
{ "message" : "Hello one" }
```

## Dockerize

```
echo 'FROM openjdk:8-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]' > Dockerfile
```

Clean, install, build, login to Docker, tag, push,
```
mvn clean install
docker image build -t helloworld .
docker run -d --name helloworld -p 8081:8080 helloworld

docker login -u <username>
docker tag helloworld remkohdev/helloworld:lab1v1.0
docker push remkohdev/helloworld:lab1v1.0
```


