# Flask App with MySQL Docker Setup
In this project, we will do containerization of a two-tier application using Docker, Docker Compose, and image scanning with Docker Scout. We will make a Flask app and a MySQL database that interact seamlessly.



![project-diagram](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/two-tier-workflow.gif)

# Prerequisites
### Before starting the project, ensure you have the following: :-
- An [AWS](https://aws.amazon.com/) account or [WSL](https://docs.microsoft.com/en-us/windows/wsl/install) installed.
- A [GitHub](https://github.com/) account.
- Code from this repository: [Clone the code](https://github.com/MayurPanchale/2-Tier-Flask-App).

## Setup Instructions

### Step 1: Launch Instance
1. **Create an AWS EC2 Instance.**
2. **Connect to the instance.** 

After successfully connecting to the EC2 instance, it will look like this
![aws-instance](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/instance.png)

### Step 2: Install Docker
 **Update the Ubuntu system:**
   ```bash
   sudo apt update
   ```
### Install docker :-
`sudo apt install docker.io`

### Now Docker is installed, but if we run the command docker ps to check for running containers, it will show an error because the current user does not have permission to connect to Docker.
![docker-error](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/docker-1.png)
### To resolve this error, add your current user to the Docker group to grant the necessary permissions. You can do this by running the following command:-
`sudo usermod -aG docker $USER`

### Reboot the system :-
`sudo reboot`
### Now if we will run docker ps it will not show any error.

## STEP 3: Clone the repository:

`git clone https://github.com/MayurPanchale/2-Tier-Flask-App.git`

![clone](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/clone-repo.png)

### Navigate to the project folder :-
`cd 2-Tier-Flask-App-and-MYSQL`
### Now do ls and you will see all files of this folder.

## Step 4: Build the Docker Image
1. **Create a Dockerfile:**
`vim Dockerfile`

   (Write your Dockerfile content)

2. **Build the Docker image: :-**
`docker build . -t flaskapp(image_name)`

3. **Verify the image:**
`docker images`

![docker-image](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/docker-image.png)

4. **Login to Docker Hub:**
`docker login`

5. **Tag and push the image to Docker Hub:**

```bash
docker tag flaskapp mayurpanchale/flaskapp:latest
```

```bash
docker push mayurpanchale(dockerhub_username)/flaskapp:latest
```
We have successfully pushed our image to the Docker registry.
![docker-repo](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/docker-repo-image.png)

*The Docker image for this application is publicly available, allowing anyone to pull and run it. You have two options to set up the application:*

1. **Create two containers and a network manually.**
2. **Use a Docker Compose file for automatic setup.**

**Choose whichever option best suits your needs!**

## Option 1: Manual Setup
### Step 1: Run the Flask Container

To create the Flask container, run:
`docker run -d -p 5000:5000 --name flaskapp flaskapp:latest`

### Step 2: Configure EC2 Security
Access the container on port 5000 by navigating to your EC2 instance’s Security Groups. Edit the inbound rules to allow traffic on port 5000.

Now, if we copy the Public IPv4 address and paste it into a new tab with Public IPv4 address:5000, our application will start running, but it may show some errors.

### Step 3: Build the MySQL Container

Next, create the MySQL container:

`docker run -d -p 3306:3306 --name mysql mysql:5.7`

### Step 4: Verify Running Containers
Check that both containers are running:
`docker ps`

### Step 5: Create a Network
Now we have to connect both containers through network. So we will create network :-
`docker network create twotier`

varify network:
`docker network ls`
![docker-network](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/network.png)

### Step 6: Connect Containers to the Network
Now, we need to assign this network to the containers. First, stop the running containers if needed(use the command: docker kill container_id), and then create new containers with the specified network using the following command: :-
```sh
docker run -d -p 5000:5000 --network twotier --name flaskapp -e MYSQL_USER=root -e MYSQL_HOST=mysql -e MYSQL_DB=KYC -e MYSQL_PASSWORD=test@123 flaskapp:latest

docker run -d --network=twotier --name mysql -e MYSQL_PASSWORD=test@123  -e MYSQL_DATABASE=KYC -e MYSQL_ROOT_PASSWORD=test@123  mysql:5.7
```
### Step 7: Inspect the Network
Ensure both containers are on the same network:
`docker network inspect twotier`

![docker-nw](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/twotier-nw.png)

Now if we do again Public IPv4 address:5000 our application show one more error.To solve this error

### Step 8: Initialize the Database
 Access the MySQL container:
`docker exec -it container_id bash`

Now, we are inside the container. If we run the ls command, it will display all directories. Next, type mysql -u root -p, enter your password (the password you set when creating the container), and you’ll be inside MySQL. The username will always be root. Once inside, type show databases;, and you will see the KYC database, which we created while setting up the container. To resolve our application error, we’ll use this database. Type use KYC; to switch to it. Now, copy the code from the message.sql file and paste it in bash.
![showdb](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/showdb.png)

This will create a table. In this table there will be 2 columns :- id and message. Now if we do again Public IPv4 address:5000 then our application will be running.

![dashboard](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/app-dashboard.png)

Now if we want to see messages so in bash type "select * from messages;" and it will show :-
![db-data](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/db-data.png)

## Option 2: Docker Compose Setup
To create multiple containers at once, we will use a Docker Compose file. First, we need to install Docker Compose. To do this, use the following command:
### Step 1: Install Docker Compose
`sudo apt install docker-compose`
### Step 2: Create a Docker Compose File
Create a docker-compose.yaml file:
`vim docker-compose.yaml`

Refer to the sample Docker Compose file in this repository or create your own.

### Step 3: Run the Docker Compose File

In this file we added volumes also because if our container distroyed somehow then our data will safe.
Start the containers with:
`docker-compose up -d`

This will automatically create the network and start all containers.

We deploy our two-tier application through docker.
## Security Scanning with Docker Scout
To check any vulnerability in image we can use docker scout.In docker CLI there will not be docker scout preintsall so we have to first install docker scout on our docker CLI. 
Create a directory for CLI plugins:
`mkdir ~/.docker/cli-plugins`
Navigate to the directory:
`cd /home/ubuntu/.docker/cli-plugins`
Now in this directory we will Install Docker Scout:
`curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s --`
Now that Docker Scout is running, if we want to scan the image we built at the beginning, run the following command
`docker scout cves sudhajobs0107/Flaskapp:latest`
And this will show us the vulnerabilities.
![scout](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/scout.png)
![scout1](https://github.com/MayurPanchale/2-Tier-Flask-App/blob/main/Images/scout1.png)

Feel free to contribute to this project or reach out for any questions. Happy learning!

## License
This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.





