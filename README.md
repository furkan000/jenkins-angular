# Angular + Jenkins + Nginx Docker CI Pipeline

This project demonstrates a CI pipeline using **Jenkins with Docker agents** to build an Angular application and package it into an Nginx Docker image. The Jenkins server itself runs in Docker and leverages the host's Docker daemon to build and manage containers.

> **Note**: This guide is intended for Linux/macOS and might vary under Windows and Docker Desktop.

---

Project Structure

```

your-project/
├── angular-app/
│   ├── package.json
│   └── src/
├── nginx-app/
│   └── Dockerfile
└── Jenkinsfile

````

---

##  Project Goals

- Automate Angular app builds using Jenkins.
- Use Docker-based agents to:
  1. Build Angular.
  2. Build an Nginx image from the compiled output.
- Keep builds reproducible using containerized tools (Node, Docker CLI).
- Avoid installing Node, Angular CLI, or Docker inside Jenkins itself.
- Reuse host-level Docker from the Jenkins container.

---

## 🧰 Requirements

Your **host machine** (Ubuntu or other Linux) must have:

- [Docker installed](https://docs.docker.com/get-docker/)
- Jenkins runs as a **Docker container** but needs access to the **host's Docker daemon** to manage containers and build images.

---

## 🚀 Jenkins Setup (Used in This Project)

We used the setup described in [this Gist](https://gist.github.com/afloesch/ea855b30cfb9f157dda8c207d40f05c0), with a simplified command for local development use.

### Start Jenkins with Docker Access

```bash
mkdir -p /var/jenkins_home

docker run -it -u root -p 8080:8080 -p 50000:50000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(which docker):/usr/bin/docker \
  -v /var/jenkins_home:/var/jenkins_home \
  --name jenkins jenkins/jenkins:lts
````

> ⚠️ **Note**: This setup is only suitable for local development and learning. 

---

## 🌐 Jenkins First-Time Setup

1. Visit [http://localhost:8080](http://localhost:8080)
2. Unlock Jenkins:

   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
3. Install Suggested Plugins
4. Create an admin user

---

## 🧪 Jenkins Pipeline

The key component of this project is the [`Jenkinsfile`](./Jenkinsfile), which defines the full CI pipeline.

### `Jenkinsfile`

```groovy
pipeline {
  agent none

  stages {
    stage('Build Angular') {
      agent {
        docker {
          image 'node:20-alpine'
        }
      }
      steps {
        sh '''
          cd angular-app
          npm install -g @angular/cli
          npm install
          ng build --configuration=production
        '''
        stash includes: 'dist/**', name: 'angular-build'
      }
    }

    stage('Build Nginx Image') {
      agent {
        docker {
          image 'docker:24.0-cli'
          args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
      }
      steps {
        unstash 'angular-build'
        sh '''
          cp -r angular-app/dist/angular-app/browser/* ./nginx-app/
          cd nginx-app
          docker build -t my-angular-nginx .
        '''
      }
    }
  }
}
```

---

## 🧱 `nginx-app/Dockerfile`

```Dockerfile
FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*
COPY . /usr/share/nginx/html/
```

This builds a lightweight image that serves your Angular app via Nginx.

---

## ✅ Running the Pipeline

1. Push your code to a Git repository.
2. Create a new **Pipeline** job in Jenkins.
3. Configure it to use your `Jenkinsfile`.
4. Run the pipeline.

---

## 🛠️ Troubleshooting

- **Linux**: Add your user to the `docker` group to avoid `sudo`:
  ```bash
  sudo usermod -aG docker $USER
  ```
  Log out and back in to apply changes.

- **Windows**: Ensure Docker Desktop is running and configured to expose the Docker daemon.


## 🗒️ Notes

* This guide is intended for Linux/macOS and might vary under Windows and Docker Desktop.
* The pipeline dynamically uses Docker agents; no tools are installed on Jenkins itself.
* Your Jenkins instance must have access to the host Docker daemon (achieved via the `-v /var/run/docker.sock:/var/run/docker.sock` mount).
