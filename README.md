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
### Logging-1
 * Create new Docker images for app:
   ```
   for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
   ```
 * Push new images in repository:
   ```
   for i in ui post comment; do docker push $USER_NAME/$i:logging; done
   ```
 * Create VM in Yandex Cloud:
   ```
   yc compute instance create \
    --name logging \
    --cores=2 \
    --memory=4GB \
    --zone ru-central1-a \
    --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
    --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=50 \
    --ssh-key ~/.ssh/id_rsa.pub
   ```
 * Create docker-machine:
   ```
   sudo docker-machine create \
    --driver generic \
    --generic-ip-address=<public_ip_VM> \
    --generic-ssh-user yc-user \
    --generic-ssh-key ~/.ssh/id_rsa \
    docker-host
   ```
 * Connect to docker-machine:
   ```
   eval $(docker-machine env logging)
   ```
 * Create compose-file named docker-compose-logging.yml
 * Create Dockerfile and build Docker image with Fluentd in directory logging/fluentd:
   ```
   docker build -t $USER_NAME/fluentd .
   ```
 * Redact docker-compose.yml in directory docker for redirect logs in Fluentd and add Zipkin
 * Run logging system in directory docker:
   ```
   docker-compose -f docker-compose-logging.yml -f docker-compose.yml down
   docker-compose -f docker-compose-logging.yml -f docker-compose.yml up -d
   ```
### Kubernetes-1
 * Create 2 pcs VM in Yandex Cloud with names master and worknode:
   ```
   yc compute instance create \
    --name <node_name> \
    --cores=4 \
    --memory=4GB \
    --zone ru-central1-a \
    --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
    --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=40,type=network-ssd \
    --ssh-key ~/.ssh/id_rsa.pub
   ```
 * Install Kubernetes on all nodes:
   ```
   sudo -i
   swapoff -a
   apt update && apt uprade
   apt install -y gnupg2 curl apt-transport-https git ca-certificates gnupg lsb-release
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
   cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
   deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
   apt update
   apt-get install -y kubelet=1.19.4-00 kubeadm=1.19.4-00 kubectl=1.19.4-00
   apt-mark hold kubelet kubeadm kubectl
   ```
 * Install Docker Engine on all nodes:
   ```
   mkdir -p /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   apt update
   apt install docker-ce=5:19.03.15~3-0~ubuntu-bionic docker-ce-cli=5:19.03.15~3-0~ubuntu-bionic containerd.io=1.3.7-1 docker-compose-plugin
   apt-mark hold docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```
 * Run on master node for cluster initialization:
   ```
   kubeadm init --apiserver-cert-extra-sans=<Public_IP_master_VM> \
                --apiserver-advertise-address=0.0.0.0 \
                --control-plane-endpoint=<Public_IP_master_VM> \
                --pod-network-cidr=10.244.0.0/16
   exit
   ```
 * Add admin config for user:
   ```
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
 * Join worknode to cluster:
   ```
   kubeadm join <Public_IP_master_VM> --token 12345 \
       --discovery-token-ca-cert-hash sha256:098764
   exit
   ```
 * Install CNI Calico:
    * Download manifest in master node:
      ```
      curl https://projectcalico.docs.tigera.io/archive/v3.22/manifests/calico.yaml -O
      ```
    * Redact calico.yaml:
      ```
            - name: CALICO_IPV4POOL_CIDR
              value: "10.244.0.0/16"
      ```
    * Deploy:
      ```
      kubectl apply -f calico.yaml
      ```
 * In directory kubernetes/reddit create manifests: post-deployment.yml, ui-deployment.yml, comment-deployment.yml, mongo-deployment.yml
 * Apply manifests:
   ```
   kubectl apply -f kubernetes/reddit
   ```
### Kubernetes-2
 * Install Debian 11 VM in local machine
 * Install kubectl:
   ```
   sudo -i
   swapoff -a
   apt update && apt uprade
   apt install -y gnupg2 curl apt-transport-https git ca-certificates gnupg software-properties-common lsb-release
   curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
   echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] \
    https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
   apt update
   apt install -y kubectl=1.19.16-00
   apt-mark hold kubectl
   ```
 * Install Docker:
   ```
   mkdir -p /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   apt update
   apt install docker-ce docker-ce-cli containerd.io
   exit
   ```
 * Install Minikube:
   ```
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
   sudo dpkg -i minikube_latest_amd64.deb
   ```
 * Add user in Docker group:
   ```
   sudo usermod -aG docker $USER && newgrp docker
   ```
 * Start Minikube:
   ```
   minikube start --kubernetes-version 1.19.16 --driver=docker
   kubectl get nodes
   ```
 * List of all contexts:
   ```
   kubectl config get-contexts
   ```
 * Watch current context:
   ```
   kubeclt config current-context
   ```
 * Select context:
   ```
   kubectl config use-context context_name
   ```
 * Create deployment manifests: comment-deployment.yml, mongo-deployment.yml, post-deployment.yml, ui-deployment.yml
 * Create service manifests: comment-mongodb-service.yml, post-mongodb-service.yml, ui-service.yml
 * Run application:
   ```
   kubectl apply -f .
   ```
 * Port forwarding for application:
   ```
   kubectl get pods --selector component=ui
   kubectl port-forward <pod-name> 8080:9292
   ```
 * In browser go to http://localhost:8080
 * Install Dashboard:
   ```
   minikube addon list
   minikube addons enable dashboard
   minikube addons enable metrics-server
   ```
 * Change ClusterIP to NodePort for service Dashboard
 * Familiarity with Dashboard:
    * Get link:
      ```
      minikube service kubernetes-dashboard -n kubernetes-dashboard
      ```
    * In browser go to <link_to_Dashboard>
 * Create and install dev namespace:
   ```
   kubectl apply -f dev-namespace.yml
   ```
 * Run application in new namespace:
   ```
   kubectl apply -n dev -f .
   minikube service ui -n dev
   ```
* Deploy Kubernetes cluster on Yandex Cloud:
   * Create Service Account:
     ```
     yc config list
     SVC_ACCT=k8s-sa
     FOLDER_ID=xxx
     yc iam service-account create --name $SVC_ACCT --folder-id $FOLDER_ID
     ACCT_ID=$(yc iam service-account get $SVC_ACCT | \
       grep ^id | \
       awk '{print $2}')
     yc resource-manager folder add-access-binding --id $FOLDER_ID \
       --role editor \
       --service-account-id $ACCT_ID
     yc iam key create --service-account-id $ACCT_ID --output ~/key/k8s-key.json
     ```
   * Create cluster through Web GUI
   * Create 2 pcs nodes
   * Connect to cluster:
     ```
     yc managed-kubernetes cluster get-credentials test-cluster --external
     kubectl config current-context
     ```
   * Create dev namespace:
     ```
     kubectl apply -f ./kubernetes/reddit/dev-namespace.yml
     ```
   * Deploy application components:
     ```
     kubectl apply -f ./kubernetes/reddit/ -n dev
     ```
### Kubernetes-3
 * Familiarity with networking in Kubernetes: ClusterIP, nodePort, LoadBalancer
 * Install Ingress Controller:
   ```
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/baremetal/deploy.yaml
   ```
 * Create Ingress for web application run command:
   ```
   kubectl apply -f ui-ingress.yml -n dev
   kubectl get ingress -A
   ```
 * Create certificate:
   ```
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=<IngressIP>"
   ```
 * Add certificate to cluster:
   ```
   kubectl create secret tls ui-ingress --key tls.key --cert tls.crt -n dev
   ```
 * Configure TLS Termination in manifest ui-ingress.yml, then run command:
   ```
   kubectl apply -f ui-ingress.yml -n dev
   ```
 * Create Secret manifest ssl-ui.yaml
 * Apply Network Policy for separation frontend and backend
 * Create virtual HDD in Yandex Cloud:
   ```
   yc compute disk create \
       --name k8s \
       --size 4 \
       --description "disk for k8s"
   ```
 * Create PersitentVolume:
   ```
   kubectl apply -f mongo-pv.yaml -n dev
   ```
 * Create PersitentVolumeClaim:
   ```
   kubectl apply -f mongo-pvc.yaml -n dev
   ```
 * Change manifest mongo-deployment.yml for attache PV and deploy DB:
   ```
   kubectl delete -f mongo-deployment.yml -n dev
   kubectl apply -f mongo-deployment.yml -n dev
   ```
### Kubernetes-4
 * Install Helm:
   ```
   wget https://get.helm.sh/helm-v3.10.1-linux-amd64.tar.gz
   tar xzvf helm-v3.10.1-linux-amd64.tar.gz
   sudo mv linux-amd64/helm /usr/local/bin/helm
   helm repo add stable https://charts.helm.sh/stable
   helm repo update
   ```
 * Create templates for components of web application
 * Install 3 pcs Ingress Controllers:
   ```
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/baremetal/deploy.yaml
   kubectl create -f ingress-controller-2.yaml
   kubectl create -f ingress-controller-3.yaml
   ```
 * Install various releases of ui:
   ```
   helm install --name test-ui-1 Chart/ui/
   helm install --name test-ui-2 Chart/ui/
   helm install --name test-ui-3 Chart/ui/
   ```
 * Install GitLab:
   ```
   helm repo add gitlab https://charts.gitlab.io
   helm fetch gitlab/gitlab --untar
   helm install gitlab . -f gitlab/values.yaml
   ```
 * Push components of web application in Gitlab
 * Setting GitLab CI and deploy web application
