## git-to-k8s-automation-pipeline

# What is this project?

This project demonstrates how to automatically build and deploy an application using:

Git (for storing code)
Jenkins (for automation)
Kubernetes (for deployment)

In simple terms:

Whenever code is updated, the system will:
Detect the change
Build the application
Deploy it automatically

No manual work needed after setup.

# Why this project exists?

In real companies, developers push code frequently.
Manually building and deploying every time is:
Slow 
Error-prone 
Not scalable 

This project solves that by creating an automated pipeline.


# How does the system work?

Let’s understand this like a real-life story, Imagine you are writing code and saving it in Git.

## how the system works (simple flow)

Step-by-step flow:

You make changes to your code
Example: fixing a bug or adding a feature
You push the code to Git
Git stores your code safely in the cloud
Git notifies Jenkins
This happens automatically using something called a webhook
Think of it like: "Hey Jenkins, new code just arrived!"
Jenkins starts working
Jenkins pulls the latest code
Builds the application (like compiling or packaging)
Runs tests (to make sure nothing is broken)
Jenkins creates a deployable artifact
Usually a Docker image (a packaged version of your app)
Jenkins sends the app to Kubernetes
Kubernetes is responsible for running your app
Kubernetes deploys the application
It makes your app live and accessible

In one line: Code push → Jenkins builds → Kubernetes deploys 

Components involved
Tool	       What it does
Git	         Stores your code
Jenkins	     Automates build & deployment
Docker	     Packages the application
Kubernetes	 Runs and manages the application

## 🛠️ Prerequisites (What you need before starting)

Before we begin setting up the CI/CD pipeline, make sure you have the following things ready.

Don’t worry — we will guide you later for each one. For now, just understand what is required.

---

### 1️⃣ A Git Account

You need a Git platform to store your code.
 
 Example:

* GitHub
* GitLab

What you should have:

* An account created
* Basic idea of pushing code

---

### 2️⃣ A Sample Application

You need some code to test the pipeline.

It can be:

* A simple Python app
* A Node.js app
* Anything that can run

Also make sure:

* Your project has a **Dockerfile** (we will create it later if not)

---

### 3️⃣ Jenkins Installed

Jenkins is the brain of our pipeline.

You need:

* Jenkins running on your system or server
* Access to Jenkins dashboard via browser

---

### 4️⃣ Docker Installed

Docker is used to package your application.

Make sure:

* Docker is installed
* You can run `docker build` and `docker run`

---

### 5️⃣ Kubernetes Cluster

This is where your app will be deployed.

 Options:

* Minikube (for local setup)
* Cloud (AWS EKS, GKE, etc.)

You should have:

* A working cluster
* `kubectl` configured

---

### 6️⃣ kubectl CLI

This tool helps you talk to Kubernetes.

Check using:

```
kubectl get nodes
```

If it shows nodes → you're good 

---

### 7️⃣ Basic Internet + Browser

You’ll need:

* Internet connection
* Browser to access Jenkins

---

## Architecture 


<img width="1174" height="609" alt="image" src="https://github.com/user-attachments/assets/101db57d-d4bc-494d-9e8c-e9424efc3ce1" />


##  Project Structure

```
.
├── app/                    # application source code
├── k8s/
│   ├── deployment.yaml     # kubernetes deployment config
│   └── service.yaml        # kubernetes service config
├── Dockerfile              # container build definition
├── Jenkinsfile             # CI/CD pipeline definition
└── README.md               # project documentation
```

---

###  Notes

* `app/` → your actual application
* `Dockerfile` → used by Jenkins to build image
* `Jenkinsfile` → defines pipeline stages
* `k8s/` → deployment configs used by kubectl

---


## 🔗 Step 1: Connect Git Repository with Jenkins

This step ensures that every code push automatically triggers Jenkins.

---

### 1️⃣ Create a new Jenkins Pipeline Job

* Open Jenkins dashboard
* Click **New Item**
* Enter name (e.g., `sample-app-pipeline`)
* Select **Pipeline**
* Click **OK**

---

### 2️⃣ Configure Git Repository

Inside the job:

* Scroll to **Pipeline section**
* Select:

  * Definition → **Pipeline script from SCM**
  * SCM → **Git**
  * Repository URL → `<your-repo-url>`
  * Branch → `main`

---

### 3️⃣ Add Credentials (if repo is private)

* Click **Add Credentials**
* Use:

  * Username + Token OR SSH key
* Select it in Jenkins

---

### 4️⃣ Enable Automatic Trigger (Webhook)

Go to:

**Job → Configure → Build Triggers**

Enable:

*  *GitHub hook trigger for GITScm polling*
  (or similar option depending on plugin)

---

### 5️⃣ Setup Webhook in Git

Go to your Git repo settings:

* Navigate to **Webhooks**
* Add new webhook

Set:

* Payload URL → `http://<jenkins-url>/github-webhook/`
* Content type → `application/json`

Save it.

---

### 🔁 What happens now?

Whenever you push code:

Git → sends webhook → Jenkins → triggers pipeline

---

###  Verification

* Make a small code change
* Push to repo
* Check Jenkins job → it should trigger automatically

---

##  Step 2: Create Jenkins Pipeline (Jenkinsfile)

This file defines the complete CI/CD flow.

Jenkins will read this file from your Git repository and execute it.

---

###  Create a file in your repo:

**File name:** `Jenkinsfile`
(no extension)

---

###  Pipeline structure

We will define stages:

1. Checkout code
2. Build Docker image
3. Push to Docker registry
4. Deploy to Kubernetes

---

###  Basic Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-docker-username/sample-app"
        TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: '<your-repo-url>'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$TAG .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:$TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }
}
```

---

###  What this pipeline does

* Pulls latest code
* Builds Docker image
* Pushes image to Docker Hub
* Deploys app to Kubernetes

---

###  Important configurations

Before running:

1. Replace:

   * `<your-repo-url>`
   * `your-docker-username`

2. Add Jenkins credentials:

   * `docker-creds` → Docker Hub login
   * `kubeconfig` → Kubernetes access file

3. Ensure:

   * Docker is installed on Jenkins
   * kubectl is configured

---

###  Output

After successful run:

* Docker image will be available in registry
* App will be deployed in Kubernetes

---

This is the **heart of the CI/CD pipeline** 

##  Step 3: Kubernetes Deployment Configuration

In this step, we define how the application should run inside Kubernetes.

We will create two files:

* Deployment → runs the application
* Service → exposes the application

---

##  1️⃣ Deployment File

Create file:

`k8s/deployment.yaml`

```yaml id="dep1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: your-docker-username/sample-app:latest
        ports:
        - containerPort: 3000
```

---

###  Key points

* `replicas: 2` → runs 2 instances of your app
* `image` → must match Docker image from Jenkins
* `containerPort` → port your app runs on

---

##  2️⃣ Service File

Create file:

`k8s/service.yaml`

```yaml id="svc1"
apiVersion: v1
kind: Service
metadata:
  name: sample-app-service
spec:
  type: NodePort
  selector:
    app: sample-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30007
```

---

###  Key points

* `type: NodePort` → exposes app outside cluster
* `port: 80` → external port
* `targetPort: 3000` → container port
* `nodePort: 30007` → access via `<node-ip>:30007`

---

## 🔁 How this connects to Jenkins

In Jenkinsfile:

```bash id="dep2"
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Jenkins uses these files to deploy/update your app

---

##  Verification

After pipeline runs:

```bash id="dep3"
kubectl get pods
kubectl get svc
```

* Pods should be running
* Service should be exposed

---

##  Result

Your application is now:

* Running inside Kubernetes
* Automatically deployed via Jenkins

---

##  Step 4: Production Considerations & Best Practices

To make this pipeline reliable and production-ready, we need to handle failures, scale properly, and follow best practices.

---

##  1️⃣ Error Handling

Ensure pipeline fails clearly and early.

### Add in Jenkinsfile:

```groovy id="err1"
options {
    failFast true
}
```

### Best practices:

* Each stage should fail if a command fails
* Avoid silent failures
* Use meaningful stage names

---

##  2️⃣ Retry Mechanism

Handle temporary issues (network, registry, etc.)

```groovy id="retry1"
stage('Push to Docker Hub') {
    steps {
        retry(2) {
            sh 'docker push $DOCKER_IMAGE:$TAG'
        }
    }
}
```

---

##  3️⃣ Notifications (Optional but useful)

Notify on success/failure.

Options:

* Email
* Slack
* MS Teams

Example (concept):

* On failure → send alert
* On success → send confirmation

---

##  4️⃣ Parameterization (Scalability)

Make pipeline reusable for multiple apps.

```groovy id="param1"
parameters {
    string(name: 'IMAGE_NAME', defaultValue: 'sample-app')
    string(name: 'NAMESPACE', defaultValue: 'default')
}
```

Use:

* Different repos
* Different environments (dev/staging/prod)

---

##  5️⃣ Security Best Practices

* Store credentials in Jenkins (never hardcode)
* Use:

  * Docker credentials
  * Kubernetes kubeconfig
* Limit access using RBAC in Kubernetes

---

##  6️⃣ Deployment Improvements

Instead of raw YAML:

 Use Helm (for advanced setups)

Benefits:

* Versioned deployments
* Easier rollbacks
* Cleaner configs

---

##  7️⃣ Monitoring & Logging

Track pipeline and app health.

Recommended:

* Jenkins logs
* Kubernetes logs:

```bash id="log1"
kubectl logs <pod-name>
```

Advanced tools:

* Prometheus (metrics)
* Grafana (visualization)

---

##  8️⃣ Rollback Strategy

In case deployment fails:

```bash id="roll1"
kubectl rollout undo deployment/sample-app
```

---

##  Final Outcome

With these improvements:

* Pipeline is stable
* Failures are handled properly
* System is scalable
* Security is maintained

---

This transforms your setup from a **basic pipeline → production-ready CI/CD system** 


## ▶️ Step 5: How to Run This Project (End-to-End)

Follow these steps to execute the complete CI/CD pipeline.

---

##  Execution Flow

### 1️⃣ Ensure everything is ready

* Jenkins is running
* Docker is installed
* Kubernetes cluster is up
* `kubectl` is configured
* Jenkins credentials are added:

  * Docker credentials
  * Kubeconfig

---

### 2️⃣ Push code to Git

Make any change in your repository and push:

```bash
git add .
git commit -m "trigger pipeline"
git push origin main
```

---

### 3️⃣ Jenkins triggers automatically

* Git webhook notifies Jenkins
* Jenkins pipeline starts

---

### 4️⃣ Pipeline execution stages

Jenkins will:

1. Checkout latest code
2. Build Docker image
3. Push image to Docker registry
4. Deploy to Kubernetes

---

### 5️⃣ Verify deployment

Run:

```bash
kubectl get pods
kubectl get svc
```

Ensure:

* Pods are running
* Service is exposed

---

### 6️⃣ Access the application

Open in browser:

```
http://<node-ip>:30007
```

 Replace `<node-ip>` with your Kubernetes node IP

---

## 🔁 What happens on next commits?

Every new push will:

* Automatically trigger Jenkins
* Rebuild and redeploy the app
* Update running version in Kubernetes

---

##  Final Result

You now have a fully working:

* Automated CI/CD pipeline
* Zero manual deployment
* Continuous delivery system

---

##  Summary

Push code → Jenkins builds → Kubernetes deploys 

---

