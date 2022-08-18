# klimenko-sergey_microservices
klimenko-sergey microservices repository

### Docker-2
 * Starting various docker containers
 * Learn how to manage docker images and containers
 * Create image from container
 * Docker machine:
   * Install the tool
   * Create VM in Yandex Cloud with docker host
   * Create Dockerfile and nessesery config files
   * Create own image:
     ```
     docker build -t reddit:latest .
     ```
   * Run container:
     ```
     docker run --name reddit -d --network=host reddit:latest
     ```
 * Download image in docker hub:
   ```
   docker tag reddit:latest klsergey/otus-reddit:1.0
   docker push klsergey/otus-reddit:1.0
   ```
 * Run own image:
   ```
   docker run --name reddit -d -p 9292:9292 klsergey/otus-reddit:1.0
   ```
### Docker-3
 * Download and unpacking archive with app:
   ```
   wget https://github.com/express42/reddit/archive/microservices.zip
   unzip microservices.zip
   mv microservices src
   cd src
   ```
 * Create Dockerfiles in directories post-py, comment, ui
 * Create Docker network:
   ```
   sudo docker network create reddit
   ```
 * Create Docker volume:
   ```
   sudo docker volume create reddit_db
   ```
 * Run containers:
   ```
   sudo docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
   sudo docker run -d --network=reddit --network-alias=post klsergey/post:1.0
   sudo docker run -d --network=reddit --network-alias=comment klsergey/comment:1.0
   sudo docker run -d --network=reddit -p 9292:9292 klsergey/ui:1.0
   ```
### Docker-4
 * Start Docker containers with various network drivers
 * Start Docker containers mongo, post, comment, ui in some network area with network aliases
 * Start Docker containers mongo, post, comment, ui in two different network areas: first area contain containers post, comment, ui; second - mongo, post, comment
 * Install Docker-compose:
   ```
   pip install docker-compose
   ```
 * Write docker-compose.yml and file .env with environment variables
 * Add environment variable "COMPOSE_PROJECT_NAME" for change project name
 * Start the project:
   ```
   docker-compose up -d
   docker-compose ps
   ```
