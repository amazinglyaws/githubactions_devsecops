
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

- now copy the public IP address of the EC2 instance and paste in the browser address bar and hit enter
    <ec2-public-ip>:9000
  
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/6405d0a7-428d-4bc8-89cb-af719e8cae51)

- provide the SonarQube login id 'admin' and password 'admin' and update the password

![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/29f517a7-ab73-4bda-b5fa-5040fa492af7)
  
- SonarQube dashboard will be displayed
  
![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/3e52d74a-e51c-4d2f-96b4-1fcad9710a3f)

#### Step 2B: Install Trivy for container vulnerability scanning

- On EC2 instance, create a trivy.sh file 
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

![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/7b44dc04-d061-4719-acff-8493fb9993ad)

- now give execute permission to trivy.sh and execute it. This will install Trivy on the EC2 instance
```
  sudo chmod +x trivy.sh
  sudo ./trivy.sh
```

- to check trivy version, run this command
```
  trivy --version
```

#### Step 3A:  Integrate SonarQube with GitHub Actions
- Go to SonarQube dashboard <EC2 public ip>:9000
- On SonarQube dashboard, select 'Manually'
- Provide project name, key and branch name and click 'Setup'
- Select 'With GitHub actions'
- Follow the steps in the Overview section to start the integration with GitHub repository
- Open your GitHub account and select your repository. In my case it is 'githubactions_devsecops'
- Click on 'Setting's and on the left pane under 'Secrets and variables' > 'Actions'
- Click 'New repository secret'
- Now go back to your SonarQube dashboard. Copy SONAR_TOKEN and click 'Generate Token'
- Click 'Generate' to generate the token
- Copy this token and click 'Continue'
- Go back to GitHub and create this secret
  ```
   Name: SONAR_TOKEN
   Secret: paste your token
  ```
- Click 'Add secret'
- Go back to SonarQube dashboard
- Copy the SONAR_HOST_URL and the URL as shown
- Go back to GitHub and create this secret
  ```
   Name: SONAR_HOST_URL
   Secret: <URL>
  ```
- Two SonarQube secrets are added in GitHub to complete the integration with GitHub. Click 'Continue'

#### Step 3B:  Create the GitHub Actions Pipeline workflow
- Select 'Other' as the Netflix project is built using React JS
- It generates a sonar properties file and a build.yml file for the pipeline workflow
- Copy the filename 'sonar-project.properties' and content 'sonar.projectKey=Netflix' as shown
  ```
   sonar-project.properties
  ```
  ```
   sonar.projectKey=Netflix
  ```
- Go back to your GitHub repo, click 'Add file' > and 'Create new file'
- Enter the filename as 'sonar-project.properties' and paste this content 'sonar.projectKey=Netflix' inside the file
- Now we need to create the pipeline workflow. To do that, click 'Add file' > Create new file' again
- Create the file name as shown
  ```
   .github/workflows/build.yaml # you may use any name for your .yml file
  ```
- Now copy the content of the build.yml file from SonarQube dashboard and paste it in the newly created GitHub file 'build.yml'
  ```
    name: Build,Analyze,scan
    
    on:
      push:
        branches:
          - main    
    
    jobs:
      build-analyze-scan:
        name: Build
        runs-on: ubuntu-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v2
            with:
              fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
    
          - name: Build and analyze with SonarQube
            uses: sonarsource/sonarqube-scan-action@master
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
              SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

  ```
- Click 'Commit changes' to create the pipeline workflow
- Click 'Actions' and it should trigger the workflow automatically
- Let's click on 'Build' and see what happens. If everything goes well, the build should complete successfully
- Now go back to SonarQube dashboard and click on 'Project' and select the project 'Netflix'
- You should see the a Summary of the SonarQube code scan. To view the full report, click on 'issues'

#### Step 3C: Add the trivy file scan in the pipeline workflow file
- Add the following code to the Build.yml file in GitHub. This will install Trivy and scan the files for any vulnerabilities. Click 'Commit changes'
  ```
    - name: install trivy
      run: |
        #install trivy
        sudo apt-get install wget apt-transport-https gnupg lsb-release -y
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
        echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy -y
        #command to scan files
        trivy fs .

  ```
-  Go back to 'Actions' and you should see the build has started
-  Analyse the build and see if the trivy file scan was completed successfully - this is another security check
  
#### Step 4: Create a TMDB API Key (to access the Netflix app)  
- Go to this url https://www.themoviedb.org/?language=en-CA and create a new account
- Now click on the profile name (on top-right) and click 'Settings'
- Click 'API' on the left pane
- Click on 'Create' and select 'Developer'
- 'Accept' the terms and conditions and provide the basic details.
- Click 'Submit' to save changes and generate the API Key. Keep this key for later use (Step 5B)

#### Step 5A: Setup the DockerHub Token 
- Login to your Dockerhub account (if you don't have, create one)
- Click profile name > Account Settings > Security and click 'New Access Token'
- Enter a name for the Access Token Description, keep the Access permissions as 'Read,Write,Delete' and click 'Generate'
- Copy and past the generated token into a text file. Click 'Copy and Close'
- Now go to GitHub and click 'Settings'. On the left pane, select 'Secrets and variables' > Actions
- Click 'New repository secret' and create the first secret as shown by clicking 'Add secret'
  ```
   Name: DOCKERHUB_USERNAME 
   Secret: enter your dockerhub username
  ```
- Create one more secret as shown
   ```
    Name: DOCKERHUB_TOKEN
    Secret: Copy and paste the GitHub token from the text file
   ```
- Click 'Add secret'

#### Step 5B: Setup the dockerbuild commands in the Build.yml pipeline workflow
- Do not forget to add the TMDB API Key (that was generated in Step 4) to the _docker build_ command 
  NOTE : do not forget to change to your dockerhub username
```
  - name: Docker build and push
    run: |
      #commands to build and push docker images
      docker build --build-arg TMDB_V3_API_KEY=<USE YOUR API KEY> -t netflix .
      docker tag netflix sunilsnair1976/netflix:latest
      docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}
      docker push sunilsnair1976/netflix:latest  
    env:
      DOCKER_CLI_ACI: 1
```
- Commit the changes
- Click on 'Actions' again and analyze the build
- If the build is successful, the docker image should be created
- Go to Dockerhub and check if the latest image is available
  
#### Step 6A: Create a self-hosted runner in EC2  
- Go to GitHub and ckick on Settings > Actions > Runners
- Click on 'New self-hosted runner'
- Select Linux and x64 Architecture
- Copy the below command to add a self-hosted runner
- Now connect to your EC2 instance using Instance Connect or SSH
- Paste the copied commands one by one
- To start the runner type
  ```
   ./run.sh
  ```
- You should see a message 'Listening for Jobs' if the runner is successfull
 ![image](https://github.com/amazinglyaws/githubactions_devsecops/assets/133778900/fc0f6ad9-7144-4a18-af68-6176c78784a0)
 
#### Step 6B: Setup the GitHub Actions pipeline to run the Netflix container  
- Add this deployment section to the Build.yml and click 'Commit changes'
  ```
    deploy:    
      needs: build-analyze-scan  
      runs-on: [aws-netflix]  
      steps:
        - name: Pull the docker image
          run: docker pull sunilsnair1976/netflix:latest
        - name: Trivy image scan
          run: trivy image sunilsnair1976/netflix:latest
        - name: Run the container netflix
          run: docker run -d --name netflix -p 8081:80 sunilsnair1976/netflix:latest
  ```
  - Now click on 'Actions'. You can see two different Jobs - 'Build and Push Docker Image' and 'deploy'
  - Click on 'Build and Push docker image' and wait for the job to complete
  - Now click on 'Summary' to go back to previous page and click 'deploy'
  - You can see the steps where the docker image is pulled from the repo and scanned by Trivy for vulnerabilities
  - This 'deploy' is running on your EC2 instance using the self-hostd runner
  - Once the deploy job is completed, you should see this successful message
  - If both the jobs are successful, you should see oth of them with a green tick symbol against their name
  - Now copy your EC2 instance public ip and enter as follows in the browser and hit enter
    ```
     <EC2-instance public ip>:8081>
    ```
      - You will see the netflix app running!
      - Deployment is completed!! Congratulations
      - Full Build.yml file
        ```
          name: Build,Analyze,scan
          
          on:
            push:
              branches:
                - main
          
          jobs:
            build-analyze-scan:
              name: Build
              runs-on: ubuntu-latest
              steps:
                - name: Checkout code
                  uses: actions/checkout@v2
                  with:
                    fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
          
                - name: Build and analyze with SonarQube
                  uses: sonarsource/sonarqube-scan-action@master
                  env:
                    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
                    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
          
                - name: install trivy
                  run: |
                     #install trivy
                     #sudo apt-get install wget apt-transport-https gnupg lsb-release -y
                     #wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
                     #echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
                     #sudo apt-get update
                     #sudo apt-get install trivy -y
                     #scanning files
                     trivy fs .
          
                - name: Docker build and push
                  run: |
                    #run commands to build and push docker images
                    docker build --build-arg TMDB_V3_API_KEY=ea34e7857cff3d57b70166e938d9cbf9 -t netflix .
                    docker tag netflix sunilsnair1976/netflix:latest
                    docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}
                    docker push sunilsnair1976/netflix:latest  
                  env:
                    DOCKER_CLI_ACI: 1     
          
            deploy:    
              needs: build-analyze-scan  
              runs-on: [aws-netflix]  
              steps:
                - name: Pull the docker image
                  run: docker pull sunilsnair1976y/netflix:latest
                - name: Trivy image scan
                  run: trivy image sunilsnair1976/netflix:latest
                - name: Run the container netflix
                  run: docker run -d --name netflix -p 8081:80 sunilsnair1976/netflix:latest
        ```
#### Step 7:  Delete the EC2 instance 
- Go to AWS console and delete the EC2 instance to avoid any billing charges

