Gestion Station Ski — CI/CD DevOps Pipeline (Jenkins • SonarQube • Nexus • Docker • Compose)

## 📌 Project Summary
This project delivers a complete CI/CD pipeline for a **Spring Boot** application named **Gestion Station Ski**.

The application:
- is built with **Spring Boot**
- uses a **SQL database (MySQL)** via Docker Compose
- is **tested with JUnit + Mockito** (unit tests with mocking)

The DevOps part:
- runs a full pipeline with **Jenkins**
- performs code quality analysis with **SonarQube**
- publishes Maven artifacts to **Nexus**
- builds and pushes a Docker image to **Docker Hub**
- deploys the stack with **Docker Compose**



## Work Performed

### Application & Testing
-  Spring Boot application packaged as a runnable `.jar`
-  SQL database integration (MySQL via Docker Compose)
-  Unit tests executed with **JUnit + Mockito**
-  Maven build lifecycle automated

### CI/CD & DevOps
-  Jenkins pipeline (Jenkinsfile)
-  Stages: checkout → clean → test → build → sonar → deploy nexus → docker build → docker push → compose deploy
-  Credentials managed securely in Jenkins (Sonar token / Nexus user / Docker Hub token)
-  Artifacts uploaded to Nexus (hosted repository)
-  Docker image versioned using: `RELEASE_VERSION-BUILD_NUMBER`

---

##  Jenkinsfile Stages (What Each Stage Does)

1. **Checkout (SCM)**
   - Jenkins pulls the project from GitHub (Pipeline script from SCM).

2. **Clean**
   - Runs `mvn clean` to reset the workspace target folders.

3. **Test**
   - Runs `mvn test` to execute unit tests.
   - Tests include **Mockito** mocks to isolate services and verify behavior.

4. **Build**
   - Packages the application as a runnable jar (`mvn install`).

5. **SonarQube Analysis**
   - Runs `mvn sonar:sonar` using a Sonar token stored in Jenkins credentials.
   - Publishes metrics: bugs, vulnerabilities, code smells, and coverage.

6. **Deploy to Nexus**
   - Publishes the built artifact to Nexus (Maven hosted repository).
   - Uses Jenkins credentials and a generated Maven `settings.xml` during the pipeline to authenticate.

7. **Build Docker Image**
   - Builds a Docker image from the produced jar.
   - Uses Java 17 base image: `eclipse-temurin:17-jre`.

8. **Push Docker Image**
   - Pushes the image to Docker Hub.
   - Uses Docker Hub token stored in Jenkins credentials.

9. **Deploy with Docker Compose**
   - Runs `docker compose down` then `docker compose up -d`
   - Starts:
     - the Spring Boot container
     - the MySQL container
   - Uses the built image (no external pull required if image is local).

---

## How We Tested the Application
### Unit Tests (JUnit + Mockito)
We validated the behavior of services/controllers with unit tests using **Mockito** to mock dependencies (repositories/services).

Run tests locally:
```bash
cd 5ARCTIC5-GestionSkieur-malekzahmoul-5arctic5
mvn test
````
Step-by-Step: What We Did (Environment → CI/CD → Deployment)
1) Prepare the Environment (Vagrant VM) : vagrant up
vagrant ssh
Install & Run DevOps Tools (Docker + Compose)

We installed Docker inside the VM and enabled it for the vagrant and jenkins users.

3) Run SonarQube + Nexus (Docker Compose)

We deployed both tools using Docker and ensured they were reachable.

SonarQube: http://localhost:9000

Nexus (host forwarded): http://localhost:8083
(inside VM it runs on http://localhost:8081)
4) Generate New Tokens / Accounts

Sonar token generated from: My Account → Security → Tokens

Nexus repo created as maven2 (hosted) (Release)

Docker Hub token created from: Account Settings → Security

5) Configure Jenkins Credentials

We created these Jenkins credentials (IDs must match Jenkinsfile):

sonar-token (Secret text)

nexus-admin-credentials (Username/Password)

docker-hub-credentials (Username/Token)

6) Create Jenkins Pipeline Job

New Item → Pipeline

Definition: Pipeline script from SCM

Repo: GitHub URL

Script Path: 5ARCTIC5-GestionSkieur-malekzahmoul-5arctic5/Jenkinsfile

7) Fix Paths & Deployment Issues

During setup we adjusted:

Running Maven commands inside the project subfolder

Nexus deployment authentication via Maven settings

Docker base image from openjdk:17 to eclipse-temurin:17-jre

Docker Hub namespace & image naming

Docker Compose to deploy using the built image

8) Run the Pipeline (Successful)

A full pipeline execution builds, tests, analyses, publishes, containerizes, pushes, and deploys the application.

How to Run the Application :
cd 5ARCTIC5-GestionSkieur-malekzahmoul-5arctic5
docker compose up -d
docker compose ps
