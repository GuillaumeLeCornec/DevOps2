# DevOps

## Terminal Commands

Follow these steps to build the Docker image, create the network, and run the containers:

### 1. Build the PostgreSQL Image

```bash
docker build -t my-postgres-db .
```

### 2. Run the PostgreSQL Container

```bash
docker run -d --name my-postgres-container my-postgres-db
```

### 3. Create the Docker Network

```bash
docker network create app-network
```

### 4. Remove the Running PostgreSQL Container

```bash
docker rm -f my-postgres-container
```

### 5. Run the PostgreSQL Container on the Network

```bash
docker run -d --name my-postgres-container --network app-network -v /data_dir:/var/lib/postgresql/data my-postgres-db
```

### 6. Run Adminer on the Network

```bash
docker run -d -p 8090:8080 --name adminer --network app-network adminer
```

### Access Adminer
Open your web browser and go to http://localhost:8090 to access Adminer. Use the following credentials to connect to the PostgreSQL database:

- Server: my-postgres-container
- User: usr
- Password: pwd
- Database: db

### Explanation

- Dockerfile: Defines the PostgreSQL image configuration, including environment variables and SQL scripts to initialize the database.
- docker build: Builds the Docker image from the Dockerfile.
- docker run: Runs a container from the built image.
- docker network create: Creates a Docker network named app-network for container communication.
- docker rm -f: Forces removal of the existing PostgreSQL container.
- docker run --network: Runs the PostgreSQL container within the created network and mounts a volume for persistent data.
- Adminer: A lightweight database management tool to manage PostgreSQL.

### Why do we need a multistage build? And explain each step of this dockerfile.

A multi-stage build in Docker is used to optimize the process of building a Docker image. It helps create smaller and more secure images by separating the build and runtime stages into different containers. 

Here are the key benefits of using a multi-stage build:

1) Reduced Image Size: Only the necessary artifacts are copied into the final image, significantly reducing the size of the Docker image.
2) Enhanced Security: By including only the dependencies needed to run the application in the final image, you reduce the potential attack surface.
3) Isolation of Build Steps: Separating the build and runtime stages ensures that build tools and development dependencies are not present in the production image.

### And explain each step of this dockerfile.

```bash
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
```
This line sets the base image for the build stage.

```bash
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
```

These lines set an environment variable MYAPP_HOME to /opt/myapp and change the working directory to this path.

```bash
COPY pom.xml .
COPY src ./src
```
These lines copy the pom.xml file and the src directory from the local machine into the Docker image.

```bash
RUN mvn package -DskipTests
```
This line runs the Maven package command to build the application, skipping tests.

```bash
FROM amazoncorretto:17
```
This line sets the base image for the run stage.

```bash
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
```
These lines set the same environment variable MYAPP_HOME and working directory as in the build stage.

```bash
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
```
This line copies the compiled JAR file from the build stage into the run stage.

```bash
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```
This line sets the entry point for the Docker container.

### Why do we need a reverse proxy?

A reverse proxy is a server that sits between client devices and a backend server, forwarding client requests to the backend server and returning the server's responses to the clients. It offers several advantages and is often used to enhance the performance, security, and manageability of web applications and services.
For this Practical work, we need it to improve our performaces.

### Why is docker-compose so important?

Docker Compose is an essential tool for managing multi-container Docker applications because it simplifies several critical aspects of deploying and managing complex environments.
The biggest benefit of Docker Compose is Multi-Container Deployment:
Docker Compose allows you to define and manage multi-container Docker applications. It enables you to deploy multiple containers, such as databases, backend services, and frontend applications, with a single command.

### 1-3 Document docker-compose most important commands.

Docker Compose provides a set of commands that help you manage multi-container applications. Here are some of the most important Docker Compose commands:

```bash
docker-compose up
```
Builds, (re)creates, starts, and attaches to containers for a service.
Options:
-d or --detach: Run containers in the background.
--build: Rebuild images before starting containers.
--scale SERVICE=NUM: Scale SERVICE to NUM instances.

```bash
docker-compose down
```

Stops and removes containers, networks, volumes, and images created by docker-compose up.
Options:
--volumes: Remove named volumes declared in the volumes section of the Compose file and anonymous volumes attached to containers.

```bash
docker-compose build
```
Builds or rebuilds services.
Options:
--no-cache: Do not use cache when building the image.
--pull: Always attempt to pull a newer version of the image.

```bash
docker-compose start
```

Starts existing containers for a service.

```bash
docker-compose stop
```

Stops running containers without removing them.


```bash
docker-compose restart
```

Restarts running containers.

```bash
docker-compose ps
```

Lists containers.
Options:
-q, --quiet: Only display IDs.

```bash
docker-compose logs
```

Views output from containers.
Options:
-f, --follow: Follow log output.

```bash
docker-compose exec
```

Runs a command in a running service container.
Options:
-T, --no-TTY: Disable pseudo-TTY allocation. By default, docker-compose exec allocates a TTY.

```bash
docker-compose run
```

Runs a one-time command against a service.
Options:
-d, --detach: Run container in the background.
--rm: Remove container after run.

### Document your docker-compose file.

```bash
services:
  API:
    container_name: API
    image: simple-api-student:latest
    environment:
      DB_host: my-postgres-container
      DB_port: 5432
      DB_name: db
      DB_user: usr
      DB_mdp: pwd
    networks:
      - app-network
    depends_on:
      - database
```

container_name: Sets a custom name for the container running this service.
image: Specifies the Docker image to use for this service (simple-api-student:latest).
environment: Defines environment variables that the service will use:
DB_host: Hostname of the database service.
DB_port: Port number for the database service.
DB_name: Name of the database.
DB_user: Username for the database.
DB_mdp: Password for the database.
networks: Connects the service to the specified network (app-network).
depends_on: Ensures that this service starts only after the database service is running.

```bash
  database:
    container_name: my-postgres-container
    image: my-postgres-db:latest
    environment:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network
```

container_name: Sets a custom name for the container running this service.
image: Specifies the Docker image to use for this service (my-postgres-db:latest).
environment: Defines environment variables for the PostgreSQL database:
POSTGRES_DB: Name of the PostgreSQL database.
POSTGRES_USER: Username for PostgreSQL.
POSTGRES_PASSWORD: Password for PostgreSQL.
volumes: Mounts a volume to persist data:
db-data:/var/lib/postgresql/data: Maps the db-data volume to the specified path in the container to ensure data persistence.
networks: Connects the service to the specified network (app-network).

```bash
  server:
    container_name: server
    image: my-running-app:latest
    environment:
      BACKEND_host: API
      BACKEND_port: "8080"
    ports:
      - "8091:80"
    networks:
      - app-network
    depends_on:
      - API
```

container_name: Sets a custom name for the container running this service.
image: Specifies the Docker image to use for this service (my-running-app:latest).
environment: Defines environment variables for the service:
BACKEND_host: Hostname of the backend API service.
BACKEND_port: Port number of the backend API service.
ports: Maps ports between the host and the container:
"8091:80": Maps port 8091 on the host to port 80 on the container.
networks: Connects the service to the specified network (app-network).
depends_on: Ensures that this service starts only after the API service is running.

```bash
networks:
  app-network:
```

networks: Defines a custom network (app-network) for the services to ensure they can communicate with each other.

```bash
volumes:
  db-data:
```

volumes: Defines a named volume (db-data) to persist data, ensuring that the data in the PostgreSQL database is not lost when the container is stopped or removed.


### 1-5 Document your publication commands and published images in dockerhub.

### Why do we put our images into an online repo?


