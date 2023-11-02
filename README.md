
# Deploy a Netflix clone using GitHub Actions powered by DevSecOps

Tools used : AWS EC2, SonarQube, Trivy, Docker, GitHub and GitHub Actions

## High Level Architecture
<img width="675" alt="image" src="https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/7a2c8df6-15cf-43a1-bf25-e54700f5a21b">

## Steps
#### Step 1:  Launch and EC2 instance (T2 micro Ubuntu Amazon AMI) 
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/66eac620-9953-49bc-ab9d-c925f54cc235)

and connect using EC2 Instance Connect (without using SSH)
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/372f737a-b47b-41bb-95eb-6aac5f2a1d60)

you should be logged in to the Ubuntu EC2 server
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/79fb29a3-6b90-45d1-85a7-3a48978b9704)


#### Step 2A: Install Docker and run SonarQube container inside EC2  
#### Step 2B: Integrate SonarQube with GitHub Actions for automated code quality checks  
#### Step 3A: Scan code files using Trivy   
#### Step 4A: Build Docker image and push to DockerHub  
#### Step 4B: Create a TMDB API Key (to access the Netflix app)  

#### Step 5A: Create a self-hosted runner in EC2  
#### Step 5B: Setup the GitHub Actions pipeline to run the Netflix container  
#### Step 6:  Delete the EC2 instance (if not free tier to incur any AWScharges)  

