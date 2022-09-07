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
 * Start Docker containers mongo, post, comment, ui in two different network areas:
    * First area contain containers post, comment, ui
    * Second - mongo, post, comment
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
### Monitoring-1
 * Create VM:
   ```
   yc compute instance create \
    --name docker-host \
    --cores=2 \
    --memory=4GB \
    --zone ru-central1-a \
    --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
    --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
    --ssh-key ~/.ssh/id_rsa.pub
   ```
 * Initialization Docker environment:
   ```
   sudo docker-machine create \
    --driver generic \
    --generic-ip-address=<IP_VM> \
    --generic-ssh-user yc-user \
    --generic-ssh-key ~/.ssh/id_rsa \
    docker-host
   ```
 * Run Prometheus - monitoring system:
   ```
   docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus
   ```
 * Create Docker image with Prometheus with custom settings in prometheus.yml file
 * Build Docker images with app in directory docker:
   ```
   for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
   ```
 * Push images in DockerHub:
   ```
   docker push klsergey/ui
   docker push klsergey/comment
   docker push klsergey/post
   docker push klsergey/prometheus
   ```
 * Add prometheus, node-exporter in section services in docker-compose.yml file
 * Add mongodb-exporter for monitoring DB
 * Add blackbox exporter for monitoring services: comment, post, ui
 * In directory docker run commands to start the project:
   ```
   docker-compose up -d
   docker-compose ps
   ```
