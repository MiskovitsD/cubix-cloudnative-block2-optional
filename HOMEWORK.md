# First task notes: modify the configuration of the __backend__ application

## Run and inspect the image

```
docker run --name backend -p 8081:8080 -d --rm quay.io/drsylent/cubix/block2/optional-backend:springboot3
docker inspect backend
```

Here I found a label and an environment variable:
> LABEL config.location=/var/opt/appconfig/application.properties
> ENV SPRING_CONFIG_LOCATION=file:///var/opt/appconfig/application.properties

## Let's copy the original configuration file

```
docker cp backend:/var/opt/appconfig/application.properties .
docker stop backend
```

I made the desired configuration file and named it `app-homework.properties`.

## Create and mount a volume to store it

```
docker volume create backend-volume
docker run --name backend -p 8081:8080 -d --rm --mount type=volume,source=backend-volume,destination=/var/opt/appconfig/homework quay.io/drsylent/cubix/block2/optional-backend:springboot3
docker cp ./app-homework.properties backend:/var/opt/appconfig/homework/app-homework.properties
docker stop backend
```

## Overwrite the label and the environment variable to point on the new configuration file

```
docker run --name backend -p 8081:8080 -d --rm --mount type=volume,source=backend-volume,destination=/var/opt/appconfig/homework --env SPRING_CONFIG_LOCATION=file:///var/opt/appconfig/homework/app-homework.properties --label config.location=/var/opt/appconfig/homework/app-homework.properties quay.io/drsylent/cubix/block2/optional-backend:springboot3
```


# Second task notes: create an image for the frontend application

## Build image, having the application built and packaged before: Dockerfile-prebuilt

```
docker build -t frontend --pull --build-arg IMAGE_BUILDER="Dániel Miskovits" -f Dockerfile-prebuilt .
```

For testing I used this command:

```
docker run --name frontend -d -p 8080:8080 --rm -e JAVA_OPTS="-Dfrontend.default.message\=env" -e JAR_ARGS="--backend.url=http://host.docker.internal:8081 --spring.main.banner-mode\=off" frontend
```

## Build Java application and Docker image: Dockerfile-multistage

```
docker build -t frontend2 --pull --build-arg IMAGE_BUILDER="Dániel Miskovits" -f Dockerfile-multistage .
```

For testing I used this command:

```
docker run --name frontend2 -d -p 8080:8080 --rm -e JAVA_OPTS="-Dfrontend.default.message\=env" -e JAR_ARGS="--backend.url=http://host.docker.internal:8081 --spring.main.banner-mode\=off" frontend2
```

# Third task notes: Docker Compose

Response from call __"Will call without message"__
```json
{
    "msForReply": 3,
    "frontendMessage": "env",
    "backendSourceData": "cubix"
}
```
