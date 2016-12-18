FROM openjdk:8

RUN useradd petclinic

RUN mkdir /app

COPY petclinic.jar /app/petclinic.jar

RUN chown -R petclinic:petclinic /app

USER petclinic

WORKDIR /app

ENTRYPOINT ["java", "-jar", "petclinic.jar"]
