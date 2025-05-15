 DevOps Task 2: 
 Jenkins CI/CD Pipeline with Docker

Objective
Set up a Jenkins pipeline to automate the process of building and deploying a Dockerized application.

 Tools & Technologies
- Jenkins (running inside Docker)
- Docker
- GitHub (as source code repository)
- EC2 (Ubuntu-based, `t2.micro`)
- Sample app (e.g., React app or Node.js app)

 Steps to Set Up

1.  Launch EC2 Instance
- Launch an Ubuntu EC2 instance (t2.micro).
- Allow **All Traffic** in the security group (for demo/testing only).
- SSH into the instance.


 2. Install Docker on EC2

sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker




3. Run Jenkins in a Docker Container


docker run -d --name c1   -p 8080:8080   -v jenkins_home:/var/jenkins_home   -v /var/run/docker.sock:/var/run/docker.sock   jenkins/jenkins:lts


Get Jenkins password:

docker exec c1 cat /var/jenkins_home/secrets/initialAdminPassword


Visit Jenkins UI: `http://<EC2-IP>:8080`


4.  Configure Jenkins
- Unlock Jenkins using the password.
- Install recommended plugins.
- Create an `admin` user.
- Install the Docker Pipeline plugin manually (if not auto-installed).
- Set Git config in Jenkins global tools.



 5.  Create a GitHub Repo
- Push your application code to GitHub.
- Add a `Jenkinsfile` in the root of the repository.

Jenkinsfile :

pipeline {
    agent any
    
    environment {
        APP_NAME = "my-react-app"
        IMAGE_TAG = "my-react-app:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building Docker Image..."
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        stage('Run container') {
            steps {
                echo "Running Docker container..."
                sh "docker rm -f ${APP_NAME} || true"
                sh "docker run -d --name ${APP_NAME} -p 3000:3000 ${IMAGE_TAG}"
            }
        }

        stage('Deploy') {
            steps {
                echo "Deployment stage (optional)..."
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}


 6.  Create a Jenkins Pipeline Job
- Go to New Item → Select Pipeline → Name it (e.g., `task2`).
- In Pipeline config:
  - Point to your GitHub repo.
  - Set branch: `*/main`
  - Script path: `Jenkinsfile`
  - 

 7.  Test the Pipeline
- Push a change to your GitHub repo.
- Jenkins auto-triggers the build.
- Watch the pipeline stages: Checkout → Build → Run → Deploy.
- Visit: `http://<EC2-IP>:3000` to see your app running.


 Outcome
 Successfully set up a CI/CD pipeline using Jenkins and Docker that:
- Automatically builds your Docker image
- Runs your app in a container
- Prepares for deployment on every code commit



- To clean up containers/images:
 
  docker container prune
  docker image prune
  

- To monitor logs:

  docker logs -f c1


