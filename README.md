
# Deploy a Netflix clone using GitHub Actions powered by DevSecOps

Tools used : AWS EC2, SonarQube, Trivy, Docker, GitHub and GitHub Actions

## High Level Architecture
<img width="675" alt="image" src="https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/7a2c8df6-15cf-43a1-bf25-e54700f5a21b">

## Steps
#### Step 1:  Launch and EC2 instance (T2 micro Ubuntu Amazon AMI) 
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/d8e69188-7686-4c8d-a072-6dab93839f89)

and connect using EC2 Instance Connect (without using SSH)
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/0c5623ff-d523-48ea-b972-eed45ce66fa4)

you should be logged in to the Ubuntu EC2 server
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/611bd636-628e-4e39-a449-0daa925eeed9)

#### Step 2A: Install Docker and run SonarQube container inside EC2  

- install docker and grant permission for 'ubuntu' user to access docker by adding it to "docker" group
    sudo apt-get update
    sudo apt install docker.io -y
    sudo usermod -aG docker ubuntu
    newgrp docker
    sudo chmod 777 /var/run/docker.sock

and check the docker installed version using command docker --version
- now create the SonarQube docker container and add port 9000 in the security group
    docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

- now copy the public IP address of the EC2 instance and past in the browser address bar and hit enter
    <ec2-public-ip>:9000

- provide the SonarQube login id and password and update the password
- SonarQube dashboard will be displayed

###### - Install docker and give permissions 
#### Step 2B: Integrate SonarQube with GitHub Actions for automated code quality checks  
#### Step 3A: Scan code files using Trivy   
#### Step 4A: Build Docker image and push to DockerHub  
#### Step 4B: Create a TMDB API Key (to access the Netflix app)  

#### Step 5A: Create a self-hosted runner in EC2  
#### Step 5B: Setup the GitHub Actions pipeline to run the Netflix container  
#### Step 6:  Delete the EC2 instance (if not free tier to incur any AWScharges)  

