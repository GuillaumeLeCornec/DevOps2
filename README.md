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

## Access Adminer
Open your web browser and go to http://localhost:8090 to access Adminer. Use the following credentials to connect to the PostgreSQL database:

- Server: my-postgres-container
- User: usr
- Password: pwd
- Database: db

## Explanation

- Dockerfile: Defines the PostgreSQL image configuration, including environment variables and SQL scripts to initialize the database.
- docker build: Builds the Docker image from the Dockerfile.
- docker run: Runs a container from the built image.
- docker network create: Creates a Docker network named app-network for container communication.
- docker rm -f: Forces removal of the existing PostgreSQL container.
- docker run --network: Runs the PostgreSQL container within the created network and mounts a volume for persistent data.
- Adminer: A lightweight database management tool to manage PostgreSQL.

## Why do we need a multistage build? And explain each step of this dockerfile.

A multi-stage build in Docker is used to optimize the process of building a Docker image. It helps create smaller and more secure images by separating the build and runtime stages into different containers. 

Here are the key benefits of using a multi-stage build:

1) Reduced Image Size: Only the necessary artifacts are copied into the final image, significantly reducing the size of the Docker image.
2) Enhanced Security: By including only the dependencies needed to run the application in the final image, you reduce the potential attack surface.
3) Isolation of Build Steps: Separating the build and runtime stages ensures that build tools and development dependencies are not present in the production image.

## Explain each step of this dockerfile.

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

## Why do we need a reverse proxy?

A reverse proxy is a server that sits between client devices and a backend server, forwarding client requests to the backend server and returning the server's responses to the clients. It offers several advantages and is often used to enhance the performance, security, and manageability of web applications and services.
For this Practical work, we need it to improve our performaces.

## Why is docker-compose so important?

Docker Compose is an essential tool for managing multi-container Docker applications because it simplifies several critical aspects of deploying and managing complex environments.
The biggest benefit of Docker Compose is Multi-Container Deployment:
Docker Compose allows you to define and manage multi-container Docker applications. It enables you to deploy multiple containers, such as databases, backend services, and frontend applications, with a single command.

## Document docker-compose most important commands.

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

## Document your docker-compose file.

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


## 1-5 Document your publication commands and published images in dockerhub.

### Step 1:  Create a Docker Hub account

Follow the link: https://hub.docker.com/

### Step 2: Tag Your Docker Images

Before pushing your images to Docker Hub, you need to tag them with your repository name and desired tag.

Tagging my-running-app:
```bash
docker tag my-running-app guillaume225/my-running-app:1.0
```

Tagging my-postgres-db:
```bash
docker tag my-postgres-db guillaume225/my-postgres-db:1.0
```

Tagging simple-api-student:
```bash
docker tag simple-api-student guillaume225/simple-api-student:1.0
```

### Step 3: Login to Docker Hub
Authenticate with Docker Hub using your credentials.

docker login -u guillaume225 -p dckr_pat_5IObyxMP3LiS1ylCmpTM6ri8C6o
Pushing my-running-app:

docker push guillaume225/my-running-app:1.0
Pushing simple-api-student:

docker push guillaume225/simple-api-student:1.0

## Why do we put our images into an online repo?

Using an online repository for Docker images, such as Docker Hub, provides several significant benefits for software development, deployment, and collaboration. For many reasons, it is a good option:

1. Collaboration and Sharing
- Team Collaboration: Online repositories allow team members to share Docker images easily. This is especially useful in collaborative environments where multiple developers or teams need access to the same images.
- Community Sharing: By pushing images to public repositories, you can share your work with the wider community. This fosters open-source collaboration and allows others to use and contribute to your projects.
2. Consistency and Reproducibility
- Standardization: Storing images in a centralized repository ensures that everyone is using the same version of an image, reducing discrepancies between different environments.
- Reproducibility: Having a standardized image available online means that you can reproduce the same environment across different machines and stages of development, testing, and production.


# TP2 - Github

## What are testcontainers?


# Document your Github Actions configurations.

The CI workflow is designed to:

- Check out the repository code.
- Set up JDK 17.
- Build and test the application using Maven.
Here is the workflow configuration:

```bash
name: CI devops 2024
on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main
      - develop

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      # Checkout your GitHub code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

      # Set up JDK 17 using actions/setup-java@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      # Change directory to where the pom.xml file is located and build/test with Maven
      - name: Build and test with Maven
        working-directory: TP1/API/simple-api-student-main
        run: mvn clean verify
```

Workflow Steps: 

1) Checkout Code: This step uses the actions/checkout@v2.5.0 action to clone the repository's code into the runner environment.

2) Set up JDK 17: This step sets up JDK 17 using the actions/setup-java@v3 action with the Temurin distribution. This ensures the correct Java version is used for building and testing the application.

3) Build and Test with Maven: This step changes the working directory to TP1/API/simple-api-student-main where the pom.xml file is located and then runs the mvn clean verify command to build and test the application.


## Document your quality gate configuration.

Here is the code I used to configure the quality gate:
```yaml 
name: CI devops 2024
on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main
      - develop

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      # Checkout your GitHub code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

      # Set up JDK 17 using actions/setup-java@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      # Build and test with Maven and SonarCloud analysis
      - name: Build, test, and analyze with SonarCloud
        working-directory: TP1/API/simple-api-student-main
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=keytp2_project-tp2 -Dsonar.organization=keytp2 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}

  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Login to Docker Hub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./TP1/API/simple-api-student-main
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/simple-api-student:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          context: ./TP1
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/my-postgres-db:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push httpd
        uses: docker/build-push-action@v3
        with:
          context: ./TP1/HTTP
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/my-running-app:latest
          push: ${{ github.ref == 'refs/heads/main' }}
```

To understand more precisly the code above, here is the part about the solarCloud configuration:

```yaml
- name: Build and test with Maven
  run: mvn -B verify sonar:sonar -Dsonar.projectKey=devops-2024 -Dsonar.organization=devops-school -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./simple-api/pom.xml
```

### Parameters
sonar.projectKey: Unique key for your SonarCloud project.
sonar.organization: Key for your SonarCloud organizatio.
sonar.host.url: URL for the SonarCloud instance.
sonar.login: Authentication token for SonarCloud, securely stored in your CI environment secrets.

### Adding the SonarCloud Token to Secrets
Go to your repository's settings.
Navigate to "Secrets" or "Variables" (depending on your CI platform).
Add a new secret named SONAR_TOKEN and set its value to your SonarCloud token.

## BONUS : DONE !
The test_backend and build-and-push-docker-image are now splited !


# TP3

## Ansible Project Setup

## Document your inventory and base commands

### SSH Key

The `id_rsa` file contains your SSH private key.

### Set Permissions for SSH Key

To set the correct permissions for your SSH private key, use the following command:

```bash
chmod 400 TP3/id_rsa
```

### Connect to the Server Manually
To manually connect to the server using SSH:

```bash
ssh -i TP3/id_rsa centos@guillaume.lecornec.takima.cloud
```
### Inventory Configuration
The setup.yml file contains the inventory configuration for Ansible:

```bash
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  children:
    prod:
      hosts: guillaume.lecornec.takima.cloud
```
### Ansible Commands
Ping the Server
To check the connectivity to the server with a ping command:

```bash
ansible all -i inventories/setup.yml -m ping
```
If everything works properly, the server should return "Pong".

### Retrieve OS Distribution
To request the server to get your OS distribution using the setup module:
```bash 
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```
### Remove Apache HTTPD Server
To remove the Apache httpd server from your machine:
```bash
ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
```

### Instructions for Use

1. Ensure you have set the correct permissions for your SSH private key using the `chmod` command.
2. Use the provided commands to connect to the server, check connectivity, retrieve OS distribution, and manage the Apache HTTPD server.


## Document your playbook

### Simple Playbook

In the same directory, in my-project/ansible/inventories, create a playbook.yml file.
To test your connection, you can add within the playbook script:

```yml
- hosts: all
  gather_facts: false
  become: true

  tasks:
   - name: Test connection
     ping:
```

Make sure that you are in the directory where you have the setup.yml and the playbook.yml files, and run this command to launch it:
```bash
ansible-playbook -i setup.yml playbook.yml
```

##  Document your playbook

### Advanced Playbook

install_docker.yml:
This playbook installs Docker and its dependencies on all targeted hosts.

Playbook Content:

```yml
- hosts: all
  gather_facts: false
  become: true

  tasks:
    - name: Install device-mapper-persistent-data
      yum:
        name: device-mapper-persistent-data
        state: latest

    - name: Install lvm2
      yum:
        name: lvm2
        state: latest

    - name: Add Docker repository
      command:
        cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

    - name: Install Docker
      yum:
        name: docker-ce
        state: present

    - name: Install python3
      yum:
        name: python3
        state: present

    - name: Install Docker SDK for Python 3
      pip:
        name: docker
        executable: pip3
      vars:
        ansible_python_interpreter: /usr/bin/python3

    - name: Ensure Docker is running
      service:
        name: docker
        state: started
      tags: docker
```
### Running the Playbook
To execute the playbook and install Docker, use the following command:
```bash
ansible-playbook -i inventories/setup.yml install_docker.yml
```
Playbook Execution Output
After running the playbook, you should see output similar to the following:
```bash
PLAY [all] ************************************************************************************************************************************************

TASK [Install device-mapper-persistent-data] **************************************************************************************************************
ok: [guillaume.lecornec.takima.cloud]

TASK [Install lvm2] ***************************************************************************************************************************************
ok: [guillaume.lecornec.takima.cloud]

TASK [Add Docker repository] ****************************************************************************************************************************
changed: [guillaume.lecornec.takima.cloud]

TASK [Install Docker] *************************************************************************************************************************************
changed: [guillaume.lecornec.takima.cloud]

TASK [Install python3] ************************************************************************************************************************************
changed: [guillaume.lecornec.takima.cloud]

TASK [Install Docker SDK for Python 3] ********************************************************************************************************************
changed: [guillaume.lecornec.takima.cloud]

TASK [Ensure Docker is running] **************************************************************************************************************************
changed: [guillaume.lecornec.takima.cloud]

PLAY RECAP ************************************************************************************************************************************************
guillaume.lecornec.takima.cloud : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
### Creating a Role for Docker Installation
To create a reusable role for Docker installation, use the following command:
```bash 
ansible-galaxy init roles/docker
```
This will generate a directory structure for the role, which you can then customize with tasks, handlers, and other necessary files for installing Docker.


## Document your docker_container tasks configuration

