
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
  ```
    sudo apt-get update
    sudo apt install docker.io -y
    sudo usermod -aG docker <user> -- my EC2 user is 'ubuntu'
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
  ```
- and check the docker installed version using command 

```
docker ps
```

![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/cc54d0ec-03cc-448f-9f86-a9c409fa4c37)

- now create the SonarQube docker container and add port 9000 in the security group
  ```
    docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
  ```
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/11455b29-3200-4c0f-96ec-91f004cdad6a)

- now copy the public IP address of the EC2 instance and past in the browser address bar and hit enter
    <ec2-public-ip>:9000
  
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/6405d0a7-428d-4bc8-89cb-af719e8cae51)

- provide the SonarQube login id 'admin' and password 'admin' and update the password

![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/29f517a7-ab73-4bda-b5fa-5040fa492af7)
  
- SonarQube dashboard will be displayed
  
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/3e52d74a-e51c-4d2f-96b4-1fcad9710a3f)

#### Step 2B: Install Trivy for container vulnerability scanning

- On EC2 server, create a trivy.sh file 
```
 sudo vi trivy.sh
```
- and paste the contents inside it. Save the file using <esc>:wq!
```
  sudo apt-get install wget apt-transport-https gnupg lsb-release -y
  wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
  echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
  sudo apt-get update
  sudo apt-get install trivy -y
```
- now given execute permission on the file and run trivy.sh
```
  sudo chmod +x trivy.sh
  sudo ./trivy.sh
```

-check trivy version
```
  trivy --version
```
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/7b44dc04-d061-4719-acff-8493fb9993ad)

#### Step 3A:  Integrate SonarQube with GitHub Actions
- go to SonarQube dashboard <EC2 public ip>:9000
- On SonarQube dashboard, select 'Manually'
- Provide project name, key and branch name and click Setup
- Select 'With GitHub actions'
- Follow the steps in the Overview section to start the integration with GitHub repository
- Open your GitHub account and select your repository. In my case it is 'githubactions_devsecops'
- Click on 'Setting's and on the left pane under 'Secrets and variables' > 'Actions'
- Click 'New rpository secret'
- Now go back to your SonarQube dashboard. Copy SONAR_TOKEN and click 'Generate Token'
- Click 'Generate' to generate the token
- Copy this token and click 'Continue'
- Go back to GitHub and create this secret
  Name: SONAR_TOKEN
  Secret: paste your token
  Click 'Add secret'
- Go back to SonarQube dasbhoard
- Copy the SONAR_HOST_URL and the URL as shown
- Go back to GitHub and create this secret
  Name: SONAR_HOST_URL
  Secret: <URL>
- Two SonarQube secrets are added in GitHub to complete the integration with GitHub. Click 'Continue'

#### Step 3B:  Create the GitHub Actions Pipeline workflow
- Select 'Other' as the Netflix project is built using React JS
- It generates a sonar properties file and a build.yml file - this is the pipeline workflow
- Copy the filename 'sonar-project.properties' and content 'sonar.projectKey=Netflix' as shown
- Go back to your GitHub repo, click 'Add file' > and 'Create new file'
- Enter the filename as 'sonar-project.properties' and past this content 'sonar.projectKey=Netflix' inside the file
- Now we need to create the pipeline workflow. To do that, click 'Add file' > Create new file' again
- Create the file name as shown .github/workflows/build.yaml -- you can use any name for the .yml file
- Now copy the content of the build.yml file from SonarQube dashboard and paste it in the newly created GitHub file 'build.yml'
- Click 'Commit changes' to create the pipeline workflow
- The workflow will be started automatically
- Let's click on 'Build' and see what happens. If everything goes well, the build should complete successfully
- Now go back to SonarQube dashboard and click on 'Project' and select the project 'Netflix'
- You can see the code scan summary. To view the full report, click on 'issues'

#### Step 3C: Add the trivy file scan in the pipeline workflow file
- Add the following code to the Build.yml file in GitHub. This will perform the file scanning. Click 'Commit changes'
  ```
  - name: run trivy filescan
    run: |
      # command to scan the files
      trivy fs .
  ```
-  Go back to 'Actions' and you should see the build has started
-  Analyse the build and see if the trivy file scan was completed successfully - this is another security check
- 
#### Step 4A: Build Docker image and push to DockerHub

#### Step 4B: Create a TMDB API Key (to access the Netflix app)  

#### Step 5:  Create a self-hosted runner in EC2  
#### Step 5B: Setup the GitHub Actions pipeline to run the Netflix container  
#### Step 6:  Delete the EC2 instance (if not free tier to incur any AWScharges)  

