

---
# A Java Maven App CI/CD with Jenkins and Kubernetes

This project demonstrates a full CI/CD pipeline for a Java Maven application.  
The pipeline is implemented in Jenkins using a `Jenkinsfile`, and it builds, packages, containerizes, and deploys the application to a Kubernetes cluster (EKS on AWS).

---

##  Pipeline Overview

The Jenkins pipeline (`Jenkinsfile`) consists of 4 stages:

1. **Build App**
   - Uses Maven to compile and package the application.
   - Generates the `.jar` file in the `target/` directory.

2. **Build Image**
   - Builds a Docker image from the packaged application.
   - Tags the image using environment variables:
     ```
     ${DOCKER_REPO}:${IMAGE_NAME}
     ```
   - Logs into AWS Elastic Container Registry (ECR).
   - Pushes the built image to ECR.

3. **Deploy**
   - Uses Kubernetes manifests (`deployment.yaml`, `service.yaml`).
   - Runs `envsubst` to substitute environment variables such as:
     - `${APP_NAME}` → Application name
     - `${DOCKER_REPO}` → AWS ECR repository URL
     - `${IMAGE_NAME}` → Build-specific Docker image tag
   - Applies the rendered manifests directly to the Kubernetes cluster with:
     ```bash
     kubectl apply -f -
     ```

4. **Commit Version Update**
   - Updates the Git repository with version bump commits.
   - Pushes changes back to the Git remote (GitLab).

---

##  Project Structure

````

├── Jenkinsfile
├── src/               # Java source code
├── Dockerfile         # Docker image build instructions
└── kubernetes/
├── deployment.yaml
└── service.yaml

````

---

## Environment Variables

The pipeline uses several environment variables and Jenkins credentials:

- `DOCKER_REPO_SERVER` → AWS ECR registry ( `381492075201.dkr.ecr.us-east-1.amazonaws.com`)
- `DOCKER_REPO` → Repository name in ECR ( `Java-maven-app`)
- `IMAGE_NAME` → Docker image tag (App version)
- `APP_NAME` → Application name (default: `java-maven-app`)
- `AWS_ACCESS_KEY_ID` & `AWS_SECRET_ACCESS_KEY` → AWS credentials (Jenkins stored secrets)
- `USER` and `PASS` → For ECR and Git authentication (Jenkins stored secrets)

---

## Docker Build Example

Locally, you can build and push the image:

```bash
docker build -t <your-repo-username>/<app-name>:<tag/version> .
docker push <your-repo-username>/<app-name>:<tag/version>
````

---

## Kubernetes Deployment Example

The kubernetes manifests has placeholders like `${APP_NAME}` and `${DOCKER_REPO}:${IMAGE_NAME}`.

You can deploy them manually (if you export the variables in your shell):

```bash
export APP_NAME=java-maven-app
export DOCKER_REPO=<your-repo-username>/<app-name>
export IMAGE_NAME=<app-name>

envsubst < kubernetes/deployment.yaml | kubectl apply -f -
envsubst < kubernetes/service.yaml | kubectl apply -f -
```

---

## Prerequisites

* Jenkins with Docker and Kubernetes plugins
* AWS ECR (Elastic Container Registry) and EKS (Elastic Kubernetes Service)
* Maven installed on the Jenkins agent
* Credentials stored in Jenkins:

  * `ecr-credentials`
  * `jenkins_aws_access_key_id`
  * `jenkins-aws_secret_access_key`
  * `github-credentials`

---

## Notes

* `envsubst` is used to dynamically replace environment variables in Kubernetes YAML files at deploy time.
* If you don’t want to use `envsubst`, you can update deployments with:

  ```bash
  kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REPO}:${IMAGE_NAME}
  ```
* The pipeline is flexible and can be extended with Helm.

---

##  Useful Commands

* Check deployment status:

  ```bash
  kubectl get deployments
  kubectl describe deployment java-maven-app
  ```

* Check service and get external IP:

  ```bash
  kubectl get svc
  ```

* Tail logs:

  ```bash
  kubectl logs -f deployment/java-maven-app
  ```

---

