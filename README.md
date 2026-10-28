# Jenkins CI/CD Pipeline with Docker

## Project Overview

This project demonstrates a Continuous Integration and Continuous Deployment (CI/CD) pipeline using Jenkins, Docker, Node.js, and Express.js.

The pipeline automates the process of checking out source code, building a Docker image, checking JavaScript syntax, deploying the application in a Docker container, and verifying that the application is running.

## Technologies Used

- **Jenkins** — CI/CD automation
- **Docker** — Application containerization
- **Node.js** — JavaScript runtime
- **Express.js** — Web application framework
- **Git and GitHub** — Version control and source code hosting
- **Linux** — Development and deployment environment

## Project Structure

```text
jenkins-cicd-task/
├── bin/
├── public/
├── routes/
├── views/
├── app.js
├── package.json
├── package-lock.json
├── Dockerfile
└── Jenkinsfile
```

## Docker Integration

The Dockerfile uses Node.js 22 Alpine as its base image, installs production dependencies, copies the application files, exposes port 3000, and starts the Express application.

Build the Docker image:

```bash
docker build -t jenkins-cicd-task:latest .
```

Run the application:

```bash
docker run -d \
  --name jenkins-cicd-app \
  -p 3001:3000 \
  jenkins-cicd-task:latest
```

Access the application at:

http://localhost:3001

## Jenkins Pipeline Stages

1. **Checkout:** Retrieves source code from GitHub.
2. **Build Docker Image:** Builds the application image.
3. **Install Dependencies:** Verifies the image and confirms dependencies were installed during the build.
4. **Code Quality:** Checks JavaScript syntax in the application files.
5. **Test:** Verifies the Node.js runtime and Docker image.
6. **Deploy:** Runs the application in a Docker container.
7. **Health Check:** Uses an HTTP request to verify that the application responds.

## Pipeline Verification

The Jenkins pipeline completed successfully with the status `SUCCESS`.

The Docker container `jenkins-cicd-app` was running with port mapping `3001:3000`, and the HTTP health check passed.

## How to Verify

Check the running container:

```bash
docker ps
```

Check the Docker image:

```bash
docker images
```

Test the application:

```bash
curl http://localhost:3001/
```

## Conclusion

This project demonstrates an automated CI/CD workflow that integrates Jenkins and Docker to build, deploy, and verify an Express.js application.
