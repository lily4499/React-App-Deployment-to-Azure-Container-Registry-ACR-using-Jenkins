
# 🚀 React App Deployment to Azure Container Registry (ACR) using Jenkins

This project demonstrates how to containerize a React app using Docker, push the image to Azure Container Registry (ACR) with a Service Principal, and automate the process using Jenkins CI pipeline.

---

## 🌍 Real-World Scenario

> At my company, our frontend team was tasked with delivering a performant and scalable React application for our cloud-based customer dashboard. To streamline deployment and eliminate manual Docker and Azure CLI steps, we containerized the app using a multi-stage Dockerfile with NGINX and automated image builds and pushes to Azure Container Registry (ACR) using Jenkins. By securely integrating a Service Principal for authentication and embedding the workflow in a Jenkins pipeline, we enabled seamless CI/CD that ensures every frontend update is reliably built, tested, and published for production in minutes.

---

## 🗂️ Project Structure

```
react-app-acr/
├── Dockerfile   #  For containerizing the app, often using NGINX to serve static files.
├── Jenkinsfile      # Defines CI/CD pipeline stages (build, test, push to registry, deploy).
├── .dockerignore    #  Lists files/folders to exclude from Docker builds (e.g., node_modules, build).
├── .gitignore       #  Lists files/folders to exclude from Git (e.g., .env, node_modules, build).
├── public/
│   └── index.html   # The HTML template React uses to inject the app.
├── src/
│   ├── App.js   # Main component, customizable for your UI.
│   └── index.js    # Entry point for the React app, renders the App component.
├── package.json      # Declares dependencies, scripts (e.g., start, build), and project metadata.
└── yarn.lock      # Tracks exact versions of installed packages if you used yarn.
```

---

## file-setup.py

```python
import os

# Define base directory
base_dir = "/home/lilia/VIDEOS/react-app-acr"

# Define file structure with content
files = {
    "Dockerfile": """\
FROM node:18-alpine AS build
WORKDIR /app
COPY . .
RUN yarn install && yarn build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
""",
    "Jenkinsfile": """\
pipeline {
    agent any

    environment {
        ACR_LOGIN_SERVER = "myacrlil.azurecr.io"
        IMAGE_NAME = "react-acr-app"
        TAG = "1"
        AZURE_TENANT_ID = 'your-tenant-id'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/yourname/react-app-acr.git'
            }
        }

        stage('Build React App') {
            steps {
                sh 'yarn install'
                sh 'yarn build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Tag & Push to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-sp-credentials', usernameVariable: 'SP_ID', passwordVariable: 'SP_SECRET')]) {
                    sh """
                    az login --service-principal -u $SP_ID -p $SP_SECRET --tenant $AZURE_TENANT_ID
                    az acr login --name myacrlil
                    docker tag ${IMAGE_NAME}:${TAG} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                    docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline complete"
        }
    }
}
""",
    ".dockerignore": "node_modules\nbuild\n.dockerignore\n.git\n.gitignore\n",
    ".gitignore": "node_modules/\nbuild/\n.env\n",
    "public/index.html": """\
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>React ACR App</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
""",
    "src/App.js": """\
import React from 'react';

function App() {
    return (
        <div>
            <h1>Hello from React ACR App!</h1>
        </div>
    );
}

export default App;
""",
    "src/index.js": """\
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
""",
    "package.json": """\
{
  "name": "react-acr-app",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-scripts": "5.0.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  }
}
""",
    "yarn.lock": "# yarn lockfile v1\n"
}

# Create files and directories
for path, content in files.items():
    file_path = os.path.join(base_dir, path)
    os.makedirs(os.path.dirname(file_path), exist_ok=True)
    with open(file_path, "w") as f:
        f.write(content)

"All files have been created in /home/lilia/VIDEOS/react-app-acr/"


```

---

## 🧱 Step-by-Step Guide

### ✅ 1. Clone the Repository

```bash
git clone https://github.com/yourname/react-app-acr.git
cd react-app-acr
```

### ✅ 2. Install Dependencies (Using Yarn)

```bash
yarn install
```

### ✅ 3. Run the App Locally

```bash
yarn start
```

Access the app in your browser at:  
➡️ http://localhost:3000

---

## 🐳 Dockerize the Application

### ✅ 4. Dockerfile

```Dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY . .
RUN yarn install && yarn build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### ✅ 5. Build Docker Image

```bash
docker build -t laly9999/react-acr-app:1 .
```

### ✅ 6. Run Container to Test

```bash
docker run -d -p 3000:80 laly9999/react-acr-app:1
```

➡️ Confirm at http://localhost:3000

---

## ☁️ Push Image to Azure Container Registry (ACR)

### ✅ 7. Create ACR & Service Principal

```bash
az group create --name myResourceGroup --location eastus
az acr create --name myacrlil --resource-group myResourceGroup --sku Basic
az ad sp create-for-rbac --name myacr-sp \
  --scopes $(az acr show --name myacrlil --query id --output tsv) \
  --role Contributor
```

✅ Save:
- SP_CLIENT_ID
- SP_CLIENT_SECRET
- TENANT_ID

### ✅ 8. Push to ACR

```bash
az login --service-principal -u <SP_CLIENT_ID> -p <SP_CLIENT_SECRET> --tenant <TENANT_ID>
az acr login --name myacrlil

docker tag laly9999/react-acr-app:1 myacrlil.azurecr.io/react-acr-app:1
docker push myacrlil.azurecr.io/react-acr-app:1
```

---

## 🤖 Jenkins CI/CD Automation

### ✅ 9. Add ACR Credentials in Jenkins

- **Kind**: Username & Password  
- **ID**: `acr-sp-credentials`  
- **Username**: `SP_CLIENT_ID`  
- **Password**: `SP_CLIENT_SECRET`

### ✅ 10. Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        ACR_LOGIN_SERVER = "myacrlil.azurecr.io"
        IMAGE_NAME = "react-acr-app"
        TAG = "1"
        AZURE_TENANT_ID = 'your-tenant-id'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/yourname/react-app-acr.git'
            }
        }

        stage('Build React App') {
            steps {
                sh 'yarn install'
                sh 'yarn build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Tag & Push to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-sp-credentials', usernameVariable: 'SP_ID', passwordVariable: 'SP_SECRET')]) {
                    sh """
                    az login --service-principal -u $SP_ID -p $SP_SECRET --tenant $AZURE_TENANT_ID
                    az acr login --name myacrlil
                    docker tag ${IMAGE_NAME}:${TAG} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                    docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline complete"
        }
    }
}
```

### ✅ 11. Create Jenkins Pipeline Job

- Go to Jenkins → New Item → Pipeline
- Name: `react-acr-pipeline`
- Paste your Jenkinsfile or pull from GitHub repo

### ✅ 12. Run Pipeline

- Click **Build Now**
- Go to Azure → ACR → Repositories → Confirm image was pushed

---

## ✅ Conclusion

You’ve successfully automated the end-to-end process of:
- Building a React app
- Dockerizing it with NGINX
- Pushing it securely to Azure Container Registry using a Service Principal
- Automating everything using Jenkins CI pipeline

Perfect for production-ready workflows in real cloud environments!

---

## 📌 Resources

- [Azure CLI Docs](https://learn.microsoft.com/en-us/cli/azure/)
- [Docker Docs](https://docs.docker.com/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
```

