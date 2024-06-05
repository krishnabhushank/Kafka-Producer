# Kafka-Producer

To create a REST endpoint for pushing messages to a Kafka topic, you can use Spring Boot along with Spring Kafka. Here's how you can set it up:

### Step-by-Step Plan

1. **Setup Spring Boot Project**:
   - Include necessary dependencies for Spring Boot and Spring Kafka.

2. **Create Kafka Producer Configuration**:
   - Configure the Kafka producer settings.

3. **Create a REST Controller**:
   - Define endpoints to accept requests and push messages to Kafka.

4. **Run the Application**:
   - Verify the endpoint works by sending HTTP requests.

### Dependencies

Add the following dependencies to your `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
</dependencies>
```

### Kafka Producer Configuration

Create a configuration class for Kafka producer settings:

```java
// Path: src/main/java/com/example/kafkaproducer/config/KafkaProducerConfig.java

package com.example.kafkaproducer.config;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.ErrorHandlingDeserializer;
import org.springframework.kafka.support.serializer.JsonSerializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

### REST Controller

Create a REST controller to handle incoming requests and push messages to Kafka:

```java
// Path: src/main/java/com/example/kafkaproducer/controller/FinancialDataController.java

package com.example.kafkaproducer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FinancialDataController {

    private final KafkaTemplate<String, String> kafkaTemplate;
    private static final String TOPIC = "financial_data";

    @Autowired
    public FinancialDataController(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    @PostMapping("/publish")
    public String publishMessage(@RequestParam("key") String key, @RequestBody String message) {
        kafkaTemplate.send(TOPIC, key, message);
        return "Message published successfully";
    }
}
```

### Main Application

Set up the main Spring Boot application class:

```java
// Path: src/main/java/com/example/kafkaproducer/KafkaProducerApplication.java

package com.example.kafkaproducer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KafkaProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(KafkaProducerApplication.class, args);
    }
}
```

### Application Properties

Configure the Kafka broker in `src/main/resources/application.properties`:

```properties
spring.kafka.bootstrap-servers=localhost:9092
```

### Running the Application

1. **Start Kafka Broker**: Make sure your Kafka broker is running on `localhost:9092`.
2. **Run the Spring Boot Application**: Use your IDE or `mvn spring-boot:run`.

### Testing the Endpoint

Use a tool like `curl` or Postman to test the endpoint:

```sh
curl -X POST -H "Content-Type: application/json" -d '{"price": "400", "quantity": "1500"}' "http://localhost:8080/publish?key=corn"
```

This should publish a message with the key "corn" to the Kafka topic "financial_data".

### Next Steps

**a.** Verify the messages in Kafka topic using Kafka consumer or CLI tools.

**b.** Add input validation and error handling for the REST endpoint.

Would you like assistance with any of these next steps or anything else?
