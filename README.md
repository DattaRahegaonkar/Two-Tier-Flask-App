 
# Flask App with MySQL Docker Setup

This is a simple Flask app that interacts with a MySQL database. The app allows users to submit messages, which are then stored in the database and displayed on the frontend.

## Basic Setup

1. Clone this repository (if you haven't already):

   ```bash
   git clone https://github.com/DattaRahegaonkar/Two-Tier-Flask-App.git
   ```

2. Navigate to the project directory:

   ```bash
   cd Two-Tier-Flask-App
   ```

3. Create a `.env` file in the project directory to store your MySQL environment variables:
```
touch .env
```

4. Open the `.env` file and add your MySQL configuration:
```
MYSQL_HOST=mysql
MYSQL_USER=your_username
MYSQL_PASSWORD=your_password
MYSQL_DB=your_database
```

## Docker Deployment

Docker installtion : sudo apt-get install docker.io (Linux)
Docker Compose : sudo apt-get install docker-compose (Linux)

1. Create a network so MySQL and Flask app can communicate:
```
docker network create twotier
```

2. Create MySQL container:
```
docker run -d \
  --name mysql \
  -v mysql-data:/var/lib/mysql \
  --network=twotier \
  -e MYSQL_DATABASE=flaskdb \
  -e MYSQL_ROOT_PASSWORD=admin \
  -p 3306:3306 \
  mysql:5.7
```

3. Build the Flask image:
```
docker build -t flaskapp .
```

4. Create Flask app container:
```
docker run -d \
  --name flaskapp \
  --network=twotier \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=admin \
  -e MYSQL_DB=flaskdb \
  -p 5000:5000 \
  flaskapp:latest
```

5. Stop and remove containers:
```
docker stop flaskapp mysql
docker rm flaskapp mysql
docker volume rm mysql-data
```

Access the flask app on http://localhost:5000


## Docker Compose Deployment

1. Create a network so MySQL and Flask app can communicate:
```
docker network create twotier
```

2. Start the containers using Docker Compose:
```
docker-compose up -d
```

Access the flask app on http://localhost:5000

3. Cleaning Up:
```
docker-compose down
```

## 🔧 Jenkins CI/CD Pipeline (DevSecOps)

This repository includes a Jenkins pipeline (Jenkinsfile) that automates the following steps:

1. Clean Workspace :
- Deletes old build artifacts to ensure a clean environment.

2. Checkout Code :
- Pulls the latest code from the GitHub repository.

3. Dependency Check (OWASP / Trivy) :
- Scans the project for security vulnerabilities in dependencies.

4. SonarQube Analysis :
- Performs code quality and security analysis using SonarQube.

5. Build Docker Image :
- Creates a Docker image for the application.

6. Push to Docker Registry :
- Pushes the image to Docker Hub (or any configured registry).

7. Deploy to Kubernetes (Kind Cluster) :
- Applies Kubernetes manifests (Deployment, Service, Ingress) to deploy the app in a local Kind cluster.

8. Email Notifications (emailext) :
- Sends automated email alerts on pipeline status (success, failure, or unstable).
- Helps the team stay informed without manually checking Jenkins.


✅ Jenkins Notes

   - Jenkins triggers can be automated via GitHub webhooks on push events.

   - Using ngrok can expose local Jenkins to GitHub for webhook testing.

   - The pipeline ensures security, quality, deployment, and notifications are automated in a single workflow.

## Notes

- Make sure to replace placeholders (e.g., `your_username`, `your_password`, `your_database`) with your actual MySQL configuration.

- This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.

- If you encounter issues, check Docker logs and error messages for troubleshooting.


![hello](https://github.com/DattaRahegaonkar/Two-Tier-Flask-App/blob/80819490ea99e8a54ad1b0a615c17420d9250a62/Jenkins%20CI-CD%20Pipeline%20.png)