The `KAFKA_BROKER` environment variable defined in the Kubernetes Deployment YAML is passed to the application container, and the Spring Boot application will use this environment variable to configure the Kafka broker list. Here’s how it works step-by-step:

### 1. Kubernetes Deployment YAML

In the Kubernetes Deployment YAML, you define the environment variable `KAFKA_BROKER`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-producer
  labels:
    app: kafka-producer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kafka-producer
  template:
    metadata:
      labels:
        app: kafka-producer
    spec:
      containers:
      - name: kafka-producer
        image: your-docker-repo/kafka-producer:latest
        ports:
        - containerPort: 8080
        env:
        - name: KAFKA_BROKER
          value: "your-kafka-broker:9092"  # Kafka broker address
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 30
```

### 2. Spring Boot Configuration

Ensure your Spring Boot application is configured to read the `KAFKA_BROKER` environment variable. This is done in your `application.properties` or `application.yml` file.

#### `application.properties`
```properties
spring.kafka.bootstrap-servers=${KAFKA_BROKER}
```

#### `application.yml`
```yaml
spring:
  kafka:
    bootstrap-servers: ${KAFKA_BROKER}
```

### 3. Dockerfile and Spring Boot Application

The Dockerfile builds the Spring Boot application into a Docker image. When the container starts, it inherits the environment variables defined in the Kubernetes Deployment YAML. The `KAFKA_BROKER` environment variable is then used by the Spring Boot application to configure the Kafka bootstrap servers.

### How It Works

1. **Kubernetes Deployment**: When the deployment is applied, Kubernetes creates pods with containers running the Spring Boot application. The environment variables defined in the `env` section of the Deployment YAML are passed to these containers.

2. **Docker Container**: The Docker container starts up and includes the environment variables defined in the Kubernetes Deployment.

3. **Spring Boot Application**: The Spring Boot application, configured to use `${KAFKA_BROKER}` in `application.properties` or `application.yml`, reads the environment variable `KAFKA_BROKER` and uses its value to set the Kafka bootstrap servers.

### Verifying the Setup

To ensure the environment variable is being passed correctly:
1. **Log the Environment Variable**: Add logging to your Spring Boot application to print the value of `spring.kafka.bootstrap-servers` on startup.
   
   ```java
   @SpringBootApplication
   public class KafkaProducerApplication {

       private static final Logger logger = LoggerFactory.getLogger(KafkaProducerApplication.class);

       @Value("${spring.kafka.bootstrap-servers}")
       private String kafkaBootstrapServers;

       public static void main(String[] args) {
           SpringApplication.run(KafkaProducerApplication.class, args);
       }

       @PostConstruct
       public void logKafkaBootstrapServers() {
           logger.info("Kafka Bootstrap Servers: {}", kafkaBootstrapServers);
       }
   }
   ```

2. **Deploy and Check Logs**: Deploy the application to your Kubernetes cluster and check the logs to verify that the correct Kafka broker list is being used.

### Next Steps:
**a.** Add logging to verify the environment variable in the application.
**b.** Deploy the application and check logs to ensure the Kafka broker list is set correctly.
