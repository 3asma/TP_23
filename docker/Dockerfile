FROM openjdk:17-jdk-slim

LABEL maintainer="Imad ADAOUMOUM"
LABEL description="Service Client - Consul Discovery"

WORKDIR /app

# Copier le JAR
COPY target/*.jar app.jar

# Exposer le port
EXPOSE 8088

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8088/actuator/health || exit 1

# Variables d'environnement par défaut
ENV SPRING_CLOUD_CONSUL_HOST=consul \
    SPRING_CLOUD_CONSUL_PORT=8500

# Démarrer l'application
ENTRYPOINT ["java", "-jar", "app.jar"]
