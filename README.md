## Prerequisites

- **Jenkins server**: Ensure you have a Jenkins server set up.
- **AWS EC2 instance**: An EC2 instance with Docker and Docker Compose installed.
- **GitHub Repository**: The source code for your Django application hosted on GitHub.

## Jenkins Pipeline Configuration

The Jenkins pipeline is defined in a Jenkinsfile, which outlines the steps for deploying the Django application.


## Pipeline Steps
- Code: Checks out the latest code from the main branch of the specified GitHub repository.
- Test Message: Displays a test message to confirm the pipeline is running.
- Deploy: Performs the following steps:
- Build Docker Image: Builds the Docker image for the Django application.
- List Docker Images: Lists the Docker images on the server.
- Run Docker Container: Runs the Docker container, mapping port 8000 on the host to port 8001 in the container.
- List Docker Containers: Lists all Docker containers to verify the deployment.

## How to Use
- Fork the repository: Fork the GitHub repository to your own account.
- Set up Jenkins: Create a new pipeline job in Jenkins and configure it to use the Jenkinsfile from your repository.
- Ensure Docker and Docker Compose are installed: Make sure your AWS EC2 instance has Docker and Docker Compose installed.
- Run the pipeline: Trigger the pipeline manually or configure it to run automatically on changes to the main branch.
