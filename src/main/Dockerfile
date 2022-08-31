FROM daocloud.io/library/java:8u40-jdk
LABEL maintainer=ll
COPY target/*.jar /usr/local/
EXPOSE 8088
WORKDIR /usr/local/
CMD java -jar demo-0.0.1-SNAPSHOT.jar
