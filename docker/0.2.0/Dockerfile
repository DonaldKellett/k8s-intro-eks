FROM ubuntu:focal
RUN apt-get update && apt-get install -y wget openjdk-11-jdk
RUN wget https://github.com/DonaldKellett/donaldsebleung-com/archive/refs/tags/v0.2.0.tar.gz && tar xvf v0.2.0.tar.gz && rm v0.2.0.tar.gz
RUN mv donaldsebleung-com-0.2.0 /app
WORKDIR /app
RUN ./mvnw package
CMD ["/usr/bin/java", "-jar", "/app/target/personal-website-0.0.1-SNAPSHOT.jar"]
