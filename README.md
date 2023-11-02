
# Deploy a Netflix clone using GitHub Actions powered by DevSecOps

Tools used : AWS EC2, SonarQube, Trivy, Docker, GitHub and GitHub Actions

## High Level Architecture
<img width="675" alt="image" src="https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/7a2c8df6-15cf-43a1-bf25-e54700f5a21b">

## Steps
#### Step 1:  Launch and EC2 instance (T2 micro Ubuntu Amazon AMI) 
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/d0b3464d-4e3a-4a4c-869a-a2992b58d9d2)


and connect using EC2 Instance Connect (or use SSH)
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/8846919d-4808-462f-8baf-da06efe48cb2)


you should be logged into the Ubuntu EC2 server
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/c19fec47-f0c2-4c08-853e-972cc4543b83)


#### Step 2A: Install Docker and run SonarQube container inside EC2  

- install docker and grant permission for 'ubuntu' user to access docker by adding it to "docker" group
    sudo apt-get update
    sudo apt install docker.io -y
    sudo usermod -aG docker ubuntu
    newgrp docker
    sudo chmod 777 /var/run/docker.sock

and check the docker installed version using command docker --version

![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/cc54d0ec-03cc-448f-9f86-a9c409fa4c37)

- now create the SonarQube docker container and add port 9000 in the security group
    docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/11455b29-3200-4c0f-96ec-91f004cdad6a)

- now copy the public IP address of the EC2 instance and past in the browser address bar and hit enter
    <ec2-public-ip>:9000
  
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/6405d0a7-428d-4bc8-89cb-af719e8cae51)

- provide the SonarQube login id 'admin' and password 'admin' and update the password

![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/29f517a7-ab73-4bda-b5fa-5040fa492af7)
  
- SonarQube dashboard will be displayed
  
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/3e52d74a-e51c-4d2f-96b4-1fcad9710a3f)

###### - Install docker and give permissions 
#### Step 2B: Integrate SonarQube with GitHub Actions for automated code quality checks  
#### Step 3A: Scan code files using Trivy   
#### Step 4A: Build Docker image and push to DockerHub  
#### Step 4B: Create a TMDB API Key (to access the Netflix app)  

#### Step 5A: Create a self-hosted runner in EC2  
#### Step 5B: Setup the GitHub Actions pipeline to run the Netflix container  
#### Step 6:  Delete the EC2 instance (if not free tier to incur any AWScharges)  

