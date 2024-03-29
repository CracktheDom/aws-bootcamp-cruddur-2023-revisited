# Week 1 — App Containerization
## Using Containerization
[Week 1 video](https://www.youtube.com/watch?v=zJnNe5Nv4tE&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=19)
* containers provide portability of code and a uniform experience across multiple system configurations.
* [linuxserver.io](https://fleet.linuxserver.io) - repository of doc files for container images
* Open Container Initiative provides standards for construction of containers
* *scratch* image is an official Docker image that acts as bedrock for containers

### Create Dockerfile for backend-flask
* Create *Dockerfile* in *backend-flask* directory and add following code:

```dockerfile
# Use the official Python 3.10 image with a minimal Debian (buster) base
FROM python:3.10-slim-buster

# Set the working directory inside the container
WORKDIR /backend-flask

# Copy the requirements.txt file from the local directory to the container
COPY requirements.txt requirements.txt

# Install Python dependencies listed in requirements.txt
RUN pip3 install -r requirements.txt

# Copy all files from the local directory to the container
COPY . .

# Set the environment variable for Flask to run in development mode
ENV FLASK_ENV=development

# Expose the port specified by the environment variable "PORT"
EXPOSE ${PORT}

# Specify the default command to run when the container starts
CMD [ "python3", "-m", "flask", "run", "--host=0.0.0.0", "--port=4567" ]
```

This Dockerfile is a set of instructions used to create a Docker image, which is essentially a standalone executable package that includes everything needed to run a piece of software. Let me break it down for you:

1. `FROM python:3.10-slim-buster`: This line specifies the base image for the Docker container. In this case, it starts with an official Python image based on version 3.10, using the "slim" variant, which is a minimalistic version, and "buster" is the Debian version.

2. `WORKDIR /backend-flask`: This sets the working directory inside the container to "/backend-flask". Subsequent commands will be executed in this directory.

3. `COPY requirements.txt requirements.txt`: Copies the "requirements.txt" file from the local directory (where the Dockerfile is located) to the "/backend-flask" directory inside the container.

4. `RUN pip3 install -r requirements.txt`: Installs Python dependencies listed in the "requirements.txt" file using the `pip3` package manager.

5. `COPY . .`: Copies all files from the local directory to the "/backend-flask" directory in the container. This includes your application code.

6. `ENV FLASK_ENV=development`: Sets an environment variable "FLASK_ENV" to "development". This is a configuration for Flask, a Python web framework, indicating that it should run in development mode.

7. `expose ${PORT}`: Exposes a port specified by the "PORT" environment variable. Note that the correct syntax is `EXPOSE` instead of `expose`.

8. `CMD [ "python3", "-m", "flask", "run", "--host=0.0.0.0", "--port=4567" ]`: Specifies the default command to run when the container starts. It runs the Flask application using the "flask run" command, listening on all available network interfaces ("0.0.0.0") and on the specified port (4567).

In summary, this Dockerfile sets up a container environment with Python 3.10, installs Flask and its dependencies, and configures the container to run a Flask application in development mode on a specified port.

#### Differences between RUN and CMD
`RUN` and `CMD` are both instructions used in a Dockerfile, but they serve different purposes:

1. **RUN:**
   - The `RUN` instruction is used to execute commands during the image-building process.
   - It is typically used for installing packages, updating the system, and performing other tasks necessary to set up the environment in the Docker image.
   - The commands specified with `RUN` are executed in a new layer on top of the current image, and the changes made during the execution are committed to the image.

   Example:
   ```dockerfile
   RUN apt-get update && apt-get install -y python
   ```

2. **CMD:**
   - The `CMD` instruction is used to provide default commands for the container that will be run when it starts.
   - It sets the default command and/or parameters, which can be overridden by providing arguments when running the container.
   - There can be only one `CMD` instruction in a Dockerfile. If multiple `CMD` instructions are specified, only the last one will take effect.

   Example:
   ```dockerfile
   CMD ["python", "app.py"]
   ```

   In this example, when the container starts, it will execute the command `python app.py`. Users can still override this command by specifying their own command when running the container.

In summary, `RUN` is used for actions during image build, while `CMD` is used to define the default command for the container when it is run.

#### Run Python Flask

* Execute the following code to start/test the Flask app: `python -m flask run --host=0.0.0.0 --port=4567`
* Navigate to http://localhost:4567/api/activities/home or http://127.0.0.1:4567/api/activities/home in a browser and "Not Found" should be displayed. In the terminal where the python command to start Flask was executed should display `404` error
* 404 error is displayed because no values were provided for environment variables when Flask ran
* Set environment variables for `FRONTEND_URL` and `BACKEND_URL` to `*`
* Restart Flask app and return to http://localhost:4567/api/activities/home, JSON response object should be displayed.
* Unset FRONTEND_URL and BACKEND_URL environment variables
* navigate to parent directory, by executing `cd ..`

#### Build container

* execute following command to build image: `docker build -t backend-flask:1.0 ./backend-flask`

  - `docker build`: This is the main command to build a Docker image.

  - `-t backend-flask:1.0`: The `-t` flag is used to tag the image with a name and optionally a tag. In this case, it tags the image with the name "backend-flask". Tags are used to identify different versions or variations of an image, i.e. "1.0".

  - `./backend-flask`: This specifies the build context, which is the location of the Dockerfile and any files it references during the build process. In this case, it points to the "backend-flask" directory, where the Dockerfile is expected to be found.

When this command is executed, Docker will look for a Dockerfile in the "backend-flask" directory, and it will use that Dockerfile to build an image. The resulting image will be tagged as "backend-flask".

#### Run the container
* execute following command to run container: `docker run --rm -p 4567:4567 -it backend-flask`

   * The `docker run` command is used to create and start a new container based on a specified Docker image. Let's break down the provided command:

   - `--rm`: This flag automatically removes the container when it exits. This is useful for temporary or disposable containers, ensuring that they don't clutter your system after they have completed their task.

   - `-p 4567:4567`: This flag maps the port from the host machine to the container. In this case, it maps port 4567 on the host to port 4567 on the container. This means that if the application inside the container is listening on port 4567, you can access it on the host machine at http://localhost:4567.

   - `-it`: This flag combines the options `-i` (interactive) and `-t` (allocate a pseudo-TTY). It allows you to interact with the container, providing an interactive terminal. This is useful for applications that require user input or when you want to manually execute commands inside the container.

   - `backend-flask`: This is the name of the Docker image to use when creating the container. In this case, it refers to the image named "backend-flask."

* Navigating to http://localhost:4567/api/activities/home will give 404 error because environment variables, `FRONTEND_URL` and `BACKEND_URL`, are not set.
* Use `-e` flag to pass environment variables to `docker` commands e.g.:
 `docker run --rm -p 4567:4567 -it ./backend-flask -e FRONTEND_URL='*' -e BACKEND_URL='*'`
* Now JSON response object will be displayed at http://localhost:4567/api/activities/home

### Create Dockerfile for frontend-react-js
* Create `Dockerfile` in the *frontend-react-js* directory
* Add following code to file:

```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD [ "npm", "start" ]
```
* `cd frontend-react-js`
* execute `npm install`
* change to parent directory, `cd ..`

#### Build image for frontend-react-js
`docker build -t frontend-react-js:1.0 ./frontend-react-js`

#### Run the container
`docker run -p 3000:3000 -d frontend-react-js:1.0`

### docker-compose.yml
* The *docker-compose.yml* file allows for the configuration of the build of multiple containers
* Create *docker-compose.yml* file in the application root directory
* Add following code to file:

```yml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "http://localhost:3000"
      BACKEND_URL: "http://localhost:4567"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "http://localhost:4567"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks:
  internal-network:
    driver: bridge
    name: cruddur
```
#### Run `docker compose`
* execute `docker compose -f "docker-compose.yml" up -d --build`

   * The `docker-compose` command is used to manage multi-container Docker applications. It reads the configuration from a `docker-compose.yml` file and creates or manages multiple containers as specified in that file.

   - `-f "docker-compose.yml"`: This flag specifies the location of the Docker Compose file that defines the services, networks, and volumes for your application. In this case, it's using a file named "docker-compose.yml."

   - `up`: This command is used to start the containers defined in the Docker Compose file. It creates and starts the services, networks, and volumes specified in the `docker-compose.yml` file.

   - `-d`: This flag stands for "detached" mode. It means that the containers will run in the background, and you'll get control of your terminal back after starting the containers. This is useful for running containers in the background without tying up your terminal.

   - `--build`: This flag tells Docker Compose to build the images before starting the services. It ensures that the images used by the services are up-to-date based on the Dockerfiles and application code. If you've made changes to your application code or Dockerfiles, using `--build` will rebuild the images.

* Navigate to http://localhost:3000 to see the Cruddur site with data pulled from backend-flask

----------------

## Create notification feature
[Week 1 video - notification](https://www.youtube.com/watch?v=k-_o0cCpksk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=25)

### openapi-3.0.yml
* `openapi-3.0.yml` file provides standard for api paths and framework
[openapis.org](https://openapis.org) - Provides details on the specifications stated the OpenAPI Specification 3.0.X
* Importing *openapi-3.0.yml* to [dash.readme.com](https://dash.readme.com) will generate documentation pages for the api
* This project was created to mirror [*Rail Service Objects*](https://www.toptal.com/ruby-on-rails/rails-service-objects-tutorial), this pattern separates functions into individual services (microservices) that allows for easier migration to the cloud, e.g. AWS API Gateway or AWS Lambda
* Structure of API response is stated in the schema outlined in *openapi-3.0.yml*
* Add api route `/api/activities/notifications` to *openapi-3.0.yml*

### backend-flask
* Add method for api route `/api/activities/notifications` to *backend-flask/app.py*
* Create new file, *notifications_activities.py*, to *backend-flask/services/* directory, and add code for class method, `NotificationsActivities`
* Build and run backend-flask container, if it is not already running
* Navigate to url for backend-flask, e.g. https://localhost:4567/api/activities/notifications to review response object

### frontend-react-js
* Entry point for React app is *frontend-react-js/src/index.js*
* in *frontend-react-js/src/App.js* add import statement for `NotificationFeedPage` and path to Notifications in the `createBrowserRouter` code section
* Create *NotificationFeedPage.js* and *NotificationFeedPage.css* files in the *frontend-react-js/src/pages/* directory
* Add code from *HomeFeedPage.css* to *NotificationFeedPage.css* and from *HomeFeedPage.js* to *NotificationFeedPage.js*, and make modifications pertinent to NotificationFeedPage route

## PostgreSQL & DynamoDB containers

### Add statements to *docker-compose.yml*

* Add following code to *docker-compose.yml* for PostgreSQL container
```yml
services:
  p-database:
  image: postgres:13-alpine
  restart: always
  environment:
    - POSTGRES_USER=postgres
    - POSTGRES_PASSWORD=password
  ports:
    - "5432:5432"
  volumes:
    - p-database:/var/lib/postgresql/data

volumes:
  p-database:
    driver: local
```

* Add following code to *docker-compose.yml* for DynamoDB container
```yml
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-data-in-volumes-lack-permission-unable-to-open-databa
    # We needs to add user:root to get this working
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```
## Test function of DynamoDB container
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html
* Start containers executing `docker compose -f "docker-compose.yml" up -d --build`

### Create DynamoDB table
```bash
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```

### Put Item into Table
```bash
aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "Men I Trust"}, "SongTitle": {"S": "Show Me How"}, "AlbumTitle": {"S": "Oncle Jazz"}}' \
    --return-consumed-capacity TOTAL
```

### List Tables
```bash
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

### Get Records
```bash
aws dynamodb scan --table-name Music --query "Items" --endpoint-url http://localhost:8000
```

## Test function of PostgreSQL container
* Display list of running containers by executing
`docker ps`
* Locate the name of the PostgreSQL container, i.e. aws-boot-cruddur-2023-revisited-p-database-1
* Then execute following command to attach a shell to the container
`docker exec -it aws-boot-cruddur-2023-revisited-p-database-1 psql -Upostgres`
* The command line interface for PostgreSQL will be displayed, `postgres=#`
* Enter `\l` to list databases
