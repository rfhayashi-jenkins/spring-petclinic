FROM openjdk:8

RUN useradd petclinic

USER petclinic

RUN mkdir /app

WORKDIR /app

COPY petclinic.war /app/petclinic.war

ENTRYPOINT ["java", "-jar", "petclinic.war"]
