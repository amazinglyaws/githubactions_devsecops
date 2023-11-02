
# Deploy a Netflix clone using GitHub Actions powered by DevSecOps

Tools used : AWS EC2, SonarQube, Trivy, Docker, GitHub and GitHub Actions

## High Level Architecture
<img width="675" alt="image" src="https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/7a2c8df6-15cf-43a1-bf25-e54700f5a21b">

## Steps
#### Step 1:  Launch and EC2 instance (T2 micro Ubuntu Amazon AMI)  
#### Step 2A: Install Docker and run SonarQube container inside EC2  
#### Step 2B: Integrate SonarQube with GitHub Actions for automated code quality checks  
#### Step 3A: Scan code files using Trivy   
#### Step 4A: Build Docker image and push to DockerHub  
#### Step 4B: Create a TMDB API Key (to access the Netflix app)  
#### Step 5A: Create a self-hosted runner in EC2  
#### Step 5B: Setup the GitHub Actions pipeline to run the Netflix container  
#### Step 6:  Delete the EC2 instance (if not free tier to incur any AWScharges)  

