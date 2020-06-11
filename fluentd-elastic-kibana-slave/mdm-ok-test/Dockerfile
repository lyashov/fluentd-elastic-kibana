FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=mdm-ok-test.jar
COPY ${JAR_FILE} app.jar
#ENTRYPOINT ["java", "-jar", "-Dhttp.proxyHost=10.0.16.240", "-Dhttps.proxyHost=10.0.16.240", "/app.jar"]
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom", "-jar","/app.jar"]


