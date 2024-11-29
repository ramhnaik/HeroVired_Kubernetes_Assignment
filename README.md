# Task:
## Create  Kubernetes deployment files and a HELM chart for a MERN (MongoDB, Express.js, React.js, Node.js) based application.

  Develop Kubernetes deployment files for both frontend and backend components, ensuring seamless deployment and scalability.    
  Create a HELM chart to streamline the deployment process, allowing for easy configuration and management.
  Write Jenkins Groovy code to automate the build and deployment process, ensuring consistency and efficiency in your CI/CD pipeline.
  The provided GitHub repositories contain the frontend and backend codebases, which you will use as the basis for your deployment configurations.
  https://github.com/UnpredictablePrashant/learnerReportCS_frontend   
  https://github.com/UnpredictablePrashant/learnerReportCS_backend

# Solution:
To achieve this, we will break down the task into three main parts:

1. **Kubernetes Deployment Files**: Create deployment files for both frontend and backend components.
2. **HELM Chart**: Create a HELM chart to streamline the deployment process.
3. **Jenkins Pipeline**: Write Jenkins Groovy code to automate the build and deployment process.

### 1. Kubernetes Deployment Files

#### Frontend Deployment (`frontend-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: unpredictableprashant/learnerreportcs_frontend:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

#### Backend Deployment (`backend-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: unpredictableprashant/learnerreportcs_backend:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  type: ClusterIP
```

### 2. HELM Chart

Create a HELM chart directory structure:
```
mychart/
  Chart.yaml
  values.yaml
  templates/
    frontend-deployment.yaml
    backend-deployment.yaml
    frontend-service.yaml
    backend-service.yaml
```

#### `Chart.yaml`
```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes
version: 0.1.0
```

#### `values.yaml`
```yaml
frontend:
  image: unpredictableprashant/learnerreportcs_frontend:latest
  replicas: 3

backend:
  image: unpredictableprashant/learnerreportcs_backend:latest
  replicas: 3
```

#### `templates/frontend-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: {{ .Values.frontend.replicas }}
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: {{ .Values.frontend.image }}
        ports:
        - containerPort: 80
```

#### `templates/backend-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: {{ .Values.backend.replicas }}
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: {{ .Values.backend.image }}
        ports:
        - containerPort: 8080
```

#### `templates/frontend-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

#### `templates/backend-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  type: ClusterIP
```

### 3. Jenkins Pipeline

#### `Jenkinsfile`
```groovy
pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'https://github.com/UnpredictablePrashant/learnerReportCS_frontend.git'
        BACKEND_REPO = 'https://github.com/UnpredictablePrashant/learnerReportCS_backend.git'
        HELM_CHART = 'mychart'
    }

    stages {
        stage('Checkout') {
            steps {
                dir('frontend') {
                    git url: "${FRONTEND_REPO}", branch: 'main'
                }
                dir('backend') {
                    git url: "${BACKEND_REPO}", branch: 'main'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'docker build -t unpredictableprashant/learnerreportcs_frontend:latest .'
                }
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'docker build -t unpredictableprashant/learnerreportcs_backend:latest .'
                }
            }
        }

        stage('Push Images') {
            steps {
                sh 'docker push unpredictableprashant/learnerreportcs_frontend:latest'
                sh 'docker push unpredictableprashant/learnerreportcs_backend:latest'
            }
        }

        stage('Deploy') {
            steps {
                sh 'helm upgrade --install myrelease ${HELM_CHART}'
            }
        }
    }
}
```
