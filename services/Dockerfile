FROM gradle:4.0-jdk8-alpine as app-builder

USER root
RUN mkdir -p /usr/services
WORKDIR /usr/services

COPY src ./src
COPY build.gradle .
RUN gradle build


# Final image with built spring boot jar and java 9 jre
FROM openjdk:8-jre-alpine

COPY --from=app-builder /usr/services/build/libs/services-0.0.1-SNAPSHOT.jar . 

ENV  JAVA_OPTS=-"Xmx3500m"
ENV  MONGODB_URI="mongodb://admin:dfgedw123X@mongodb:27017/admin"
ENV  MINIO_URL="http://minio:9000"
ENV  MINIO_ACCESS_KEY="58CZ5TR5PZV5RX14Q6I8"
ENV  MINIO_SECRET_KEY="JdY2rppyyCK1cEJ+tH8Cy7WhPAqkp1to2y/e2J4R"
ENV  RABBITMQ_HOST="rabbitmq"
ENV  RABBITMQ_STOMP_PORT="61613"
ENV  RABBITMQ_PORT="5672"
ENV  RABBITMQ_USER="rabbit"
ENV  RABBITMQ_PASS="wek30dlbT56"
ENV  SKIP_RULE_PROCESSING_FOR_EMPTY_DATASET_PROPERTY="true"
ENV  SKIP_EMPTY_GENERATED_VALUES="true"
ENV  IMPORT_DEMO_SESSION="true"
ENV  PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin"
ENV  LANG="C.UTF-8"
ENV  JAVA_HOME="/usr/lib/jvm/java-1.8-openjdk/jre"
ENV  JAVA_VERSION="8u151"
ENV  JAVA_ALPINE_VERSION="8.151.12-r0"
EXPOSE 8083

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar ./services-0.0.1-SNAPSHOT.jar"]