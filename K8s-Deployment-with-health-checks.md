Sure, adding the Spring Boot Actuator endpoints will allow Kubernetes to perform more sophisticated health checks. We'll update the Deployment YAML to include the necessary liveness and readiness probes.

Here is the updated Deployment YAML:

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
          value: "your-kafka-broker:9092"
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

### Explanation:
1. **Liveness Probe:**
   - Path: `/actuator/health/liveness`
   - Delay: 30 seconds after container startup.
   - Period: Every 30 seconds.

2. **Readiness Probe:**
   - Path: `/actuator/health/readiness`
   - Delay: 30 seconds after container startup.
   - Period: Every 30 seconds.

### Assumptions:
- Your Spring Boot application exposes the `/actuator/health/liveness` and `/actuator/health/readiness` endpoints.

### Next Steps:
**a.** Configure Spring Boot Actuator to expose custom liveness and readiness endpoints.
**b.** Test the health checks locally before deploying to the Kubernetes cluster.
