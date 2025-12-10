# Complete One-Day DevOps Tutorial
## From LAMP Stack to Full DevOps Pipeline

**Time Required:** 8-10 hours  
**Prerequisites:** Ubuntu Server with LAMP stack installed, sudo access

---

## Hour 1-2: Git Mastery & Docker Fundamentals

### 1.1 Git Setup and Best Practices

```bash
# Install/verify Git
sudo apt update
sudo apt install git -y
git --version

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main

# Create a sample PHP project
mkdir ~/devops-demo && cd ~/devops-demo
git init

# Create a simple PHP application
cat > index.php << 'EOF'
<?php
phpinfo();
echo "<h1>DevOps Demo - Version 1.0</h1>";
echo "<p>Environment: " . getenv('APP_ENV') . "</p>";
?>
EOF

cat > config.php << 'EOF'
<?php
$db_host = getenv('DB_HOST') ?: 'localhost';
$db_user = getenv('DB_USER') ?: 'root';
$db_pass = getenv('DB_PASS') ?: '';
$db_name = getenv('DB_NAME') ?: 'demo';
?>
EOF

# Create .gitignore
cat > .gitignore << 'EOF'
*.log
.env
vendor/
node_modules/
EOF

# First commit
git add .
git commit -m "Initial commit: PHP application"

# Create development branch
git checkout -b develop
```

### 1.2 Install Docker

```bash
# Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc

# Install prerequisites
sudo apt install ca-certificates curl gnupg lsb-release -y

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y

# Add your user to docker group (logout/login required)
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker compose version
```

---

## Hour 2-3: Dockerize Your Application

### 2.1 Create Dockerfile

```bash
cd ~/devops-demo

cat > Dockerfile << 'EOF'
FROM php:8.2-apache

# Install PHP extensions
RUN docker-php-ext-install mysqli pdo pdo_mysql

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Copy application files
COPY . /var/www/html/

# Set permissions
RUN chown -R www-data:www-data /var/www/html

EXPOSE 80

CMD ["apache2-foreground"]
EOF
```

### 2.2 Create Docker Compose Configuration

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    build: .
    container_name: devops-web
    ports:
      - "8080:80"
    environment:
      - APP_ENV=development
      - DB_HOST=db
      - DB_USER=devops_user
      - DB_PASS=devops_pass
      - DB_NAME=devops_db
    volumes:
      - .:/var/www/html
    depends_on:
      - db
    networks:
      - devops-network

  db:
    image: mysql:8.0
    container_name: devops-db
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=devops_db
      - MYSQL_USER=devops_user
      - MYSQL_PASSWORD=devops_pass
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - devops-network

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: devops-phpmyadmin
    ports:
      - "8081:80"
    environment:
      - PMA_HOST=db
      - PMA_USER=root
      - PMA_PASSWORD=rootpass
    depends_on:
      - db
    networks:
      - devops-network

volumes:
  db-data:

networks:
  devops-network:
    driver: bridge
EOF
```

### 2.3 Build and Run

```bash
# Build and start containers
docker compose up -d

# Check running containers
docker compose ps

# View logs
docker compose logs -f web

# Test the application
curl http://localhost:8080

# Access in browser:
# - App: http://your-server-ip:8080
# - phpMyAdmin: http://your-server-ip:8081
```

### 2.4 Commit Docker Configuration

```bash
git add Dockerfile docker-compose.yml
git commit -m "Add Docker configuration"
```

---

## Hour 3-4: Jenkins CI/CD Setup

### 3.1 Install Jenkins with Docker

```bash
# Create Jenkins directory
mkdir -p ~/jenkins_home

# Run Jenkins container
docker run -d \
  --name jenkins \
  -p 8082:8080 -p 50000:50000 \
  -v ~/jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(which docker):/usr/bin/docker \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Access Jenkins at: http://your-server-ip:8082
```

### 3.2 Jenkins Initial Setup

1. Open browser to `http://your-server-ip:8082`
2. Enter the initial admin password
3. Install suggested plugins
4. Create admin user
5. Install additional plugins:
   - Docker Pipeline
   - Git plugin
   - Pipeline plugin

### 3.3 Create Jenkins Pipeline

Create a file called `Jenkinsfile` in your project:

```bash
cat > Jenkinsfile << 'EOF'
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "devops-demo"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'file:///home/ubuntu/devops-demo/.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo "Running tests..."
                    sh """
                        docker run --rm ${DOCKER_IMAGE}:${DOCKER_TAG} \
                        php -l /var/www/html/index.php
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying application..."
                    sh """
                        docker compose down || true
                        docker compose up -d
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
EOF

git add Jenkinsfile
git commit -m "Add Jenkins pipeline"
```

### 3.4 Create Jenkins Job

1. Click "New Item" in Jenkins
2. Enter name: "devops-demo-pipeline"
3. Select "Pipeline" and click OK
4. Under Pipeline section:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: `file:///home/ubuntu/devops-demo/.git`
   - Branch: `*/main`
5. Save and click "Build Now"

---

## Hour 4-5: Ansible Configuration Management

### 4.1 Install Ansible

```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y

# Verify installation
ansible --version
```

### 4.2 Create Ansible Structure

```bash
cd ~/devops-demo
mkdir -p ansible/{roles,inventory,playbooks}

# Create inventory
cat > ansible/inventory/hosts << 'EOF'
[webservers]
localhost ansible_connection=local

[all:vars]
ansible_python_interpreter=/usr/bin/python3
EOF
```

### 4.3 Create Ansible Playbook for Docker Deployment

```bash
cat > ansible/playbooks/deploy.yml << 'EOF'
---
- name: Deploy DevOps Demo Application
  hosts: webservers
  become: yes
  vars:
    app_dir: /home/ubuntu/devops-demo
    
  tasks:
    - name: Ensure Docker is installed
      apt:
        name:
          - docker.io
          - docker-compose
        state: present
        update_cache: yes
      
    - name: Ensure Docker service is running
      service:
        name: docker
        state: started
        enabled: yes
    
    - name: Pull latest code
      git:
        repo: "file://{{ app_dir }}/.git"
        dest: "{{ app_dir }}/deployment"
        version: main
        force: yes
      become: no
    
    - name: Stop existing containers
      command: docker compose down
      args:
        chdir: "{{ app_dir }}"
      ignore_errors: yes
    
    - name: Build and start containers
      command: docker compose up -d --build
      args:
        chdir: "{{ app_dir }}"
    
    - name: Wait for application to be ready
      wait_for:
        port: 8080
        delay: 5
        timeout: 60
    
    - name: Verify application is running
      uri:
        url: http://localhost:8080
        status_code: 200
      register: app_status
    
    - name: Display deployment status
      debug:
        msg: "Application deployed successfully!"
      when: app_status.status == 200
EOF
```

### 4.4 Run Ansible Playbook

```bash
# Test connection
ansible -i ansible/inventory/hosts webservers -m ping

# Run deployment playbook
ansible-playbook -i ansible/inventory/hosts ansible/playbooks/deploy.yml

# Add to git
git add ansible/
git commit -m "Add Ansible configuration"
```

---

## Hour 5-6: Monitoring with Prometheus & Grafana

### 5.1 Add Monitoring to Docker Compose

```bash
cat >> docker-compose.yml << 'EOF'

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - devops-network

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - prometheus
    networks:
      - devops-network

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    networks:
      - devops-network

volumes:
  db-data:
  prometheus-data:
  grafana-data:
EOF
```

### 5.2 Create Prometheus Configuration

```bash
cat > prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'docker-containers'
    static_configs:
      - targets: ['web:80']
EOF
```

### 5.3 Restart with Monitoring

```bash
docker compose down
docker compose up -d

# Access monitoring tools:
# Prometheus: http://your-server-ip:9090
# Grafana: http://your-server-ip:3000 (admin/admin)
```

### 5.4 Configure Grafana

1. Open Grafana at `http://your-server-ip:3000`
2. Login with admin/admin (change password when prompted)
3. Add Prometheus data source:
   - Configuration > Data Sources > Add data source
   - Select Prometheus
   - URL: `http://prometheus:9090`
   - Click "Save & Test"
4. Import Node Exporter Dashboard:
   - Dashboards > Import
   - Enter ID: 1860
   - Select Prometheus data source
   - Click Import

---

## Hour 6-7: Kubernetes Basics with K3s

### 6.1 Install K3s (Lightweight Kubernetes)

```bash
# Install K3s
curl -sfL https://get.k3s.io | sh -

# Check status
sudo systemctl status k3s

# Set up kubectl access
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config

# Verify installation
kubectl get nodes
kubectl get pods -A
```

### 6.2 Create Kubernetes Manifests

```bash
mkdir -p k8s

# Create namespace
cat > k8s/namespace.yml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: devops-demo
EOF

# Create deployment
cat > k8s/deployment.yml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-web
  namespace: devops-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-web
  template:
    metadata:
      labels:
        app: devops-web
    spec:
      containers:
      - name: web
        image: devops-demo:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
        env:
        - name: APP_ENV
          value: "production"
        - name: DB_HOST
          value: "mysql-service"
EOF

# Create service
cat > k8s/service.yml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: devops-web-service
  namespace: devops-demo
spec:
  type: NodePort
  selector:
    app: devops-web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
EOF
```

### 6.3 Deploy to Kubernetes

```bash
# Import Docker image to K3s
docker save devops-demo:latest | sudo k3s ctr images import -

# Apply manifests
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/deployment.yml
kubectl apply -f k8s/service.yml

# Check deployment
kubectl get pods -n devops-demo
kubectl get services -n devops-demo

# Access application
curl http://localhost:30080

# Add to git
git add k8s/
git commit -m "Add Kubernetes manifests"
```

---

## Hour 7-8: Complete CI/CD Pipeline Integration

### 7.1 Enhanced Jenkinsfile with K8s Deployment

```bash
cat > Jenkinsfile << 'EOF'
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "devops-demo"
        DOCKER_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = "/home/ubuntu/.kube/config"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    sh """
                        docker run --rm ${DOCKER_IMAGE}:${DOCKER_TAG} \
                        php -l /var/www/html/index.php
                    """
                }
            }
        }
        
        stage('Push to K3s Registry') {
            steps {
                script {
                    sh """
                        docker save ${DOCKER_IMAGE}:latest | sudo k3s ctr images import -
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl apply -f k8s/namespace.yml
                        kubectl apply -f k8s/deployment.yml
                        kubectl apply -f k8s/service.yml
                        kubectl rollout restart deployment/devops-web -n devops-demo
                        kubectl rollout status deployment/devops-web -n devops-demo
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sh """
                        sleep 10
                        curl -f http://localhost:30080 || exit 1
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "✓ Pipeline completed successfully!"
            echo "Application available at: http://localhost:30080"
        }
        failure {
            echo "✗ Pipeline failed! Check logs for details."
        }
        always {
            sh "docker system prune -f"
        }
    }
}
EOF

git add Jenkinsfile
git commit -m "Update Jenkins pipeline with K8s deployment"
```

### 7.2 Create Automated Deployment Script

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash

echo "===== DevOps Automated Deployment ====="

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Function to print colored output
print_status() {
    echo -e "${GREEN}[✓]${NC} $1"
}

print_error() {
    echo -e "${RED}[✗]${NC} $1"
}

print_info() {
    echo -e "${YELLOW}[i]${NC} $1"
}

# Check prerequisites
print_info "Checking prerequisites..."
command -v docker >/dev/null 2>&1 || { print_error "Docker not installed"; exit 1; }
command -v kubectl >/dev/null 2>&1 || { print_error "kubectl not installed"; exit 1; }
print_status "Prerequisites met"

# Build Docker image
print_info "Building Docker image..."
docker build -t devops-demo:latest . || { print_error "Build failed"; exit 1; }
print_status "Docker image built"

# Import to K3s
print_info "Importing image to K3s..."
docker save devops-demo:latest | sudo k3s ctr images import - || { print_error "Import failed"; exit 1; }
print_status "Image imported to K3s"

# Deploy to Kubernetes
print_info "Deploying to Kubernetes..."
kubectl apply -f k8s/ || { print_error "Deployment failed"; exit 1; }
print_status "Deployed to Kubernetes"

# Wait for rollout
print_info "Waiting for deployment..."
kubectl rollout status deployment/devops-web -n devops-demo || { print_error "Rollout failed"; exit 1; }
print_status "Deployment complete"

# Verify
print_info "Verifying deployment..."
sleep 5
curl -f http://localhost:30080 > /dev/null 2>&1 || { print_error "Verification failed"; exit 1; }
print_status "Application is running"

echo ""
print_status "===== Deployment Successful ====="
echo ""
echo "Access points:"
echo "  Application: http://localhost:30080"
echo "  Prometheus: http://localhost:9090"
echo "  Grafana: http://localhost:3000"
echo "  Jenkins: http://localhost:8082"
EOF

chmod +x deploy.sh
git add deploy.sh
git commit -m "Add automated deployment script"
```

---

## Hour 8: Testing & Documentation

### 8.1 Create Health Check Script

```bash
cat > healthcheck.sh << 'EOF'
#!/bin/bash

echo "===== DevOps Stack Health Check ====="
echo ""

# Check Docker
echo "Docker Containers:"
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
echo ""

# Check Kubernetes
echo "Kubernetes Pods:"
kubectl get pods -n devops-demo
echo ""

# Check Services
echo "Testing Services:"
services=("8080:Docker Web" "8082:Jenkins" "9090:Prometheus" "3000:Grafana" "30080:K8s Web")

for service in "${services[@]}"; do
    port="${service%%:*}"
    name="${service#*:}"
    if curl -s -o /dev/null -w "%{http_code}" http://localhost:$port | grep -q "200\|302"; then
        echo "✓ $name (port $port): OK"
    else
        echo "✗ $name (port $port): FAILED"
    fi
done

echo ""
echo "===== Health Check Complete ====="
EOF

chmod +x healthcheck.sh
```

### 8.2 Run Complete Health Check

```bash
./healthcheck.sh
```

### 8.3 Create README Documentation

```bash
cat > README.md << 'EOF'
# DevOps Demo Project

Complete DevOps pipeline demonstration with Docker, Jenkins, Ansible, Kubernetes, Prometheus, and Grafana.

## Architecture

```
Source Code (Git)
    ↓
Jenkins CI/CD Pipeline
    ↓
Docker Build
    ↓
Kubernetes Deployment (K3s)
    ↓
Monitoring (Prometheus + Grafana)
```

## Quick Start

```bash
# Deploy everything
./deploy.sh

# Check health
./healthcheck.sh
```

## Access Points

- **Application (Docker)**: http://localhost:8080
- **Application (K8s)**: http://localhost:30080
- **Jenkins**: http://localhost:8082
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)
- **phpMyAdmin**: http://localhost:8081

## Manual Deployment

### Docker Compose
```bash
docker compose up -d
```

### Kubernetes
```bash
kubectl apply -f k8s/
```

### Ansible
```bash
ansible-playbook -i ansible/inventory/hosts ansible/playbooks/deploy.yml
```

## Monitoring

- View metrics in Prometheus: http://localhost:9090
- Grafana dashboards: http://localhost:3000
- Import Node Exporter dashboard (ID: 1860)

## CI/CD Pipeline

Pipeline automatically:
1. Checks out code from Git
2. Builds Docker image
3. Runs tests
4. Deploys to Kubernetes
5. Verifies deployment

## Maintenance Commands

```bash
# View logs
docker compose logs -f
kubectl logs -f -l app=devops-web -n devops-demo

# Restart services
docker compose restart
kubectl rollout restart deployment/devops-web -n devops-demo

# Scale deployment
kubectl scale deployment/devops-web --replicas=3 -n devops-demo

# Clean up
docker compose down
kubectl delete namespace devops-demo
```

## Technologies Used

- **Docker & Docker Compose**: Containerization
- **Jenkins**: CI/CD automation
- **Ansible**: Configuration management
- **Kubernetes (K3s)**: Container orchestration
- **Prometheus**: Metrics collection
- **Grafana**: Visualization
- **Git**: Version control

## Project Structure

```
devops-demo/
├── ansible/
│   ├── inventory/
│   └── playbooks/
├── k8s/
│   ├── namespace.yml
│   ├── deployment.yml
│   └── service.yml
├── index.php
├── config.php
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── prometheus.yml
├── deploy.sh
└── healthcheck.sh
```

## Next Steps

1. Add automated testing
2. Implement blue-green deployment
3. Set up log aggregation with ELK
4. Add security scanning
5. Implement GitOps with ArgoCD
6. Add Helm charts
7. Implement service mesh (Istio)

## Troubleshooting

### Container won't start
```bash
docker compose logs web
docker compose restart
```

### Kubernetes pod issues
```bash
kubectl describe pod <pod-name> -n devops-demo
kubectl logs <pod-name> -n devops-demo
```

### Jenkins build fails
- Check Jenkins logs: `docker logs jenkins`
- Verify Docker socket permissions
- Ensure kubectl access is configured

## License

MIT
EOF

git add README.md healthcheck.sh
git commit -m "Add documentation and health check"
```

---

## Summary & Next Steps

### What You've Accomplished Today:

1. ✓ **Version Control**: Git repository with branches
2. ✓ **Containerization**: Docker and Docker Compose
3. ✓ **CI/CD**: Jenkins pipeline with automated builds
4. ✓ **Configuration Management**: Ansible playbooks
5. ✓ **Monitoring**: Prometheus and Grafana
6. ✓ **Orchestration**: Kubernetes (K3s) deployment
7. ✓ **Automation**: Complete deployment scripts

### Your DevOps Stack:

```
Application Layer:     PHP/Apache
Database Layer:        MySQL
Container Runtime:     Docker
Orchestration:         Kubernetes (K3s)
CI/CD:                Jenkins
Config Management:     Ansible
Monitoring:           Prometheus + Grafana
Version Control:      Git
```

### Quick Reference Commands:

```bash
# Deploy everything
./deploy.sh

# Health check
./healthcheck.sh

# View all services
docker ps
kubectl get all -n devops-demo

# Trigger Jenkins build
# Visit: http://localhost:8082

# View metrics
# Visit: http://localhost:9090 (Prometheus)
# Visit: http://localhost:3000 (Grafana)
```

### Practice Exercises:

1. **Modify the application** and watch the CI/CD pipeline deploy it
2. **Scale the Kubernetes deployment** to 5 replicas
3. **Create custom Grafana dashboard** for your application
4. **Add a new Ansible task** to install monitoring agents
5. **Implement rolling updates** in Kubernetes

### Advanced Topics to Explore:

- **Helm**: Package manager for Kubernetes
- **ArgoCD**: GitOps continuous delivery
- **Terraform**: Infrastructure as code
- **ELK Stack**: Elasticsearch, Logstash, Kibana for logs
- **Vault**: Secrets management
- **Istio**: Service mesh
- **Trivy/Clair**: Container security scanning

### Resources for Continued Learning:

- Docker Documentation: https://docs.docker.com
- Kubernetes Documentation: https://kubernetes.io/docs
- Jenkins Handbook: https://www.jenkins.io/doc/book
- Ansible Documentation: https://docs.ansible.com
- Prometheus Documentation: https://prometheus.io/docs

---

## Congratulations!

You've built a complete DevOps pipeline from scratch in one day. This hands-on experience covers the core tools and practices used in modern DevOps environments. Keep practicing, experimenting, and building on this foundation!

**Pro Tip**: Commit your changes regularly and experiment with breaking things - that's how you learn troubleshooting skills!

```bash
# Push to remote repository (if you have one)
git remote add origin <your-repo-url>
git push -u origin main
```