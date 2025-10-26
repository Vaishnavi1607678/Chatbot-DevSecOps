# Chatbot-DevSecOps

# PROJECT GOAL

You will build and run a **DevSecOps pipeline locally** that:

* Clones the Chatbot UI app
* Runs SonarQube code quality check
* Runs OWASP Dependency Check
* Builds and runs Docker container
* Scans image with Trivy
* Deploys and serves chatbot at **[http://localhost:3000](http://localhost:3000)**

---

# 📁 FOLDER STRUCTURE

Create a working folder on your local system:

```
Chatbot-DevSecOps/
│
├── docker-compose.yml
├── Jenkinsfile
├── README.md
└── chatbot-ui/        
```

---

# STEP 1: Install Required Tools

Run everything below one-by-one (Linux/WSL):

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose curl unzip nodejs npm openjdk-17-jdk git
sudo systemctl start docker
sudo systemctl enable docker
```

Verify:

```bash
docker --version
docker-compose --version
java -version
node -v
npm -v
```

---
<img width="1920" height="1080" alt="Screenshot (685)" src="https://github.com/user-attachments/assets/7dc22dce-5f7d-4b7d-ab97-e725805a8221" />

# STEP 2: Clone Chatbot UI Source Code

```bash
cd ~/Chatbot-DevSecOps
git clone https://github.com/NotHarshhaa/DevOps-Projects.git
mv DevOps-Projects/DevOps-Project-28/Chatbot-UI ./chatbot-ui
rm -rf DevOps-Projects
```

---

# STEP 3: Create `docker-compose.yml`

This file will run **Jenkins**, **SonarQube**, and **Chatbot UI** containers together.

```yaml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always

  sonarqube:
    image: sonarqube:lts
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      - SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true
    restart: always

  chatbot-ui:
    build: ./chatbot-ui
    container_name: chatbot-ui
    ports:
      - "3000:3000"
    restart: always

volumes:
  jenkins_home:
```

---

<img width="1920" height="1080" alt="Screenshot (688)" src="https://github.com/user-attachments/assets/33ba8699-1de2-4c87-ae2e-e14e275c57a4" />

<img width="1920" height="1080" alt="Screenshot (687)" src="https://github.com/user-attachments/assets/3e6626ee-6cae-4aa7-b977-035f669ee322" />

<img width="1920" height="1080" alt="Screenshot (686)" src="https://github.com/user-attachments/assets/8a08f877-a2b2-498e-86eb-75d07c716b5c" />

<img width="1920" height="1080" alt="Screenshot (690)" src="https://github.com/user-attachments/assets/16fb713d-73ee-43ee-bf58-5cfa5a36d99f" />

# STEP 4: Build and Run All Containers

```bash
docker-compose up -d --build
```

Check running containers:

```bash
docker ps
```

Expected:

```
CONTAINER ID   IMAGE              PORTS                     NAMES
xxxxxx         chatbot-ui         0.0.0.0:3000->3000/tcp    chatbot-ui
xxxxxx         sonarqube:lts      0.0.0.0:9000->9000/tcp    sonarqube
xxxxxx         jenkins/jenkins    0.0.0.0:8080->8080/tcp    jenkins
```

---
<img width="1920" height="1080" alt="Screenshot (691)" src="https://github.com/user-attachments/assets/510cf4f9-a203-416b-b659-84e5052cb353" />

<img width="1920" height="1080" alt="Screenshot (689)" src="https://github.com/user-attachments/assets/1ef066cc-0132-4876-865e-9282c151e179" />

<img width="1920" height="1080" alt="Screenshot (693)" src="https://github.com/user-attachments/assets/8f9ca07e-dcca-4511-a34d-0df836b4cb05" />

<img width="1920" height="1080" alt="Screenshot (694)" src="https://github.com/user-attachments/assets/e9d2dad5-e9eb-457b-8e0b-3bcc0061bc5c" />

# STEP 5: Configure Jenkins

Visit Jenkins → [http://localhost:8080](http://localhost:8080)
Get password:

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

1. Install **Suggested Plugins**
2. Create **Admin User**
3. Go to **Manage Jenkins → Plugins → Available Plugins**
   Install these:

   * Docker
   * NodeJS
   * SonarQube Scanner
   * OWASP Dependency Check
   * Trivy plugin
   * Pipeline Utility Steps

---

# STEP 6: Configure Jenkins Tools

Navigate → **Manage Jenkins → Tools**

Add:

* **JDK 17**
* **NodeJS 19**
* **SonarQube Scanner**
* **Dependency Check (DP-Check)**

---
<img width="1920" height="1080" alt="Screenshot (692)" src="https://github.com/user-attachments/assets/6bfd3423-7fd1-4e58-bd78-3823c59a4d6f" />

<img width="1920" height="1080" alt="Screenshot (698)" src="https://github.com/user-attachments/assets/f6cc15e8-3f10-4b93-9025-2926cae0f3b5" />

<img width="1920" height="1080" alt="Screenshot (696)" src="https://github.com/user-attachments/assets/6a5265dc-8781-4862-a8b9-2ca16ba433e9" />

# STEP 7: Configure SonarQube

Go to [http://localhost:9000](http://localhost:9000)
Login: `admin / admin`
Change password.

Now:

* Go to → **My Account → Security → Generate Token**
* Copy the token (example: `sonar123token`)

Back in Jenkins:

* **Manage Jenkins → Credentials → Global**

  * Add “Secret Text” → ID: `sonar-token`
  * Value: `your SonarQube token`

Then go to:
**Manage Jenkins → Configure System → SonarQube Servers**

Add:

```
Name: sonar-server
Server URL: http://sonarqube:9000
Token: sonar-token
```

Click Apply & Save.

---

# STEP 8: Create the Jenkinsfile

Create a file in your main folder:

```bash
nano Jenkinsfile
```

Paste this **error-free local Jenkins pipeline** 👇

```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node19'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/NotHarshhaa/DevOps-Projects.git', branch: 'main'
                dir('DevOps-Project-28/Chatbot-UI') {
                    echo "Checked out Chatbot UI source"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('DevOps-Project-28/Chatbot-UI') {
                    sh 'npm install'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    dir('DevOps-Project-28/Chatbot-UI') {
                        sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=ChatbotUI \
                        -Dsonar.projectName=ChatbotUI
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dir('DevOps-Project-28/Chatbot-UI') {
                    dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }

        stage('Docker Build & Run') {
            steps {
                dir('DevOps-Project-28/Chatbot-UI') {
                    script {
                        sh 'docker build -t chatbot-ui .'
                        sh 'docker stop chatbot-ui || true'
                        sh 'docker rm chatbot-ui || true'
                        sh 'docker run -d -p 3000:3000 --name chatbot-ui chatbot-ui'
                    }
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh 'trivy image chatbot-ui > trivy_report.txt || true'
            }
        }
    }
}
```

---

# STEP 9: Create Jenkins Pipeline Job

1. In Jenkins dashboard → “New Item”
2. Name: `Chatbot-UI-Local`
3. Type: **Pipeline**
4. Definition → “Pipeline script from SCM”
5. SCM: Git
   URL: `https://github.com/NotHarshhaa/DevOps-Projects.git`
   Branch: `main`
   Script Path: `DevOps-Project-28/Chatbot-UI/Jenkinsfile`
6. Save → Build Now 

---

# STEP 10: Test Application

After build succeeds:

```bash
docker ps
```

Output:

```
chatbot-ui     0.0.0.0:3000->3000/tcp   chatbot-ui
```

Visit 👉 [http://localhost:3000](http://localhost:3000)
Scroll to bottom → Paste your **OpenAI API key** → click Save.
Now chat with your local AI chatbot 

---

# STEP 11: (Optional) View Reports

* SonarQube dashboard → [http://localhost:9000](http://localhost:9000)
* OWASP & Trivy reports → in Jenkins workspace
* Jenkins console output → shows pipeline logs

---
<img width="1920" height="1080" alt="Screenshot (697)" src="https://github.com/user-attachments/assets/72aec284-7e7b-4056-acba-6a5a2d1f016c" />

# STEP 12: Clean Up

To stop all containers:

```bash
docker-compose down
```

To remove everything:

```bash
docker system prune -a --volumes -f
```

---

# FINAL OUTPUT SUMMARY

| Service     | URL                                            | Description          |
| ----------- | ---------------------------------------------- | -------------------- |
| Jenkins     | [http://localhost:8080](http://localhost:8080) | CI/CD automation     |
| SonarQube   | [http://localhost:9000](http://localhost:9000) | Code quality scanner |
| Chatbot UI  | [http://localhost:3000](http://localhost:3000) | Deployed app         |
| Trivy/OWASP | Jenkins reports                                | Security scans       |

---

