### Initial Download & Account Creation

- download docker - https://docs.docker.com/get-started/get-docker/
- Create Account with docker (remember for later)- https://app.docker.com/signup?
- Create GCP account (most likely use google account) - https://cloud.google.com/?hl=en
- Download gcp - https://cloud.google.com/sdk/docs/install
- Create Account on aws - https://aws.amazon.com/

![Click console to get to screen where you need to work](./images/Screenshot%202024-09-25%20172445.png)

Click console to get to screen where you need to work

### Login on Command Line/Terminal

Open up your terminal of choosing 

1. Log in to your dockers 

```powershell
docker login
```

2. Set up your GCP

```powershell
 gcloud init
```

- It should then open a page to login with google in browser, which you should click the account you want to use and give permissions
- You eventually want to go back to the terminal, and it should ask for you to select/create project, which you want to do, I already had one created  named “  launchcodecapstonezachary”

To verify your logged in and your project you put the following commands in 

```powershell
gcloud auth list
```

 and 

```powershell
 gcloud projects list
```

---

### Create Your Database using AWS & Connect to MySQL

1. You want to start on the following page

![Screenshot 2024-09-25 182805.png](./images/Screenshot%202024-09-25%20182805.png)

2. In the search bar, type "rds" and select the corresponding option

![Screenshot 2024-09-25 182927.png](./images/Screenshot%202024-09-25%20182927.png)

3. Click databases on the left side and click the orange create database

![Screenshot 2024-09-25 183007.png](./images/Screenshot%202024-09-25%20183007.png)

4. You want to change the following things on the database, then click create database at the end
- database creation - **standard**
- engine options -  **MySQL**
- Templates -  **Free tier**
- Settings choose your **master username**, make sure **credentials** are self managed and enter a master password that has some strength to it!

![Screenshot 2024-09-25 183558.png](./images/Screenshot%202024-09-25%20183558.png)

- **Connectivity** - **Public Access** should have **Yes**

![Screenshot 2024-09-25 183917.png](./images/Screenshot%202024-09-25%20183917.png)

**You will then have to wait a few minutes for the database to be created**

5. Once created, click on it, and copy then note down the Endpoint & port. 

![Screenshot 2024-09-25 184309.png](./images/Screenshot%202024-09-25%20184309.png)

6. Then scroll down until you see the security group, click the security group that has a type of **EC2 Security Group**

![Screenshot 2024-09-25 184316.png](./images/Screenshot%202024-09-25%20184316.png)

It should take you to the following page, which will automatically filter for only one. Click the security group ID link: 

![Screenshot 2024-09-25 184720.png](./images/Screenshot%202024-09-25%20184720.png)

7. The page should look something like the image below. Click "Edit inbound rules."

![Screenshot 2024-09-25 184939.png](./images/Screenshot%202024-09-25%20184939.png)

8. Add a rule, which takes a custom TCP, Port Range of 3306, and click the Source dropdown to select "Anywhere-IPv4"

Typically, allowing access from all IPv4 addresses is not recommended for security reasons. However, given our specific situation, it should be acceptable. If you're concerned about enhancing security, be aware that it may involve additional costs. You might find tutorials online that offer guidance on improving security measures.

![Screenshot 2024-09-25 185101.png](./images/Screenshot%202024-09-25%20185101.png)

**Click the orange "Save rules" button.**

9. Go to MySQL Workbench, add a server connection with the information you were told to note down and test the connection to make sure everything is working fine

![Screenshot 2024-09-25 185509.png](./images/Screenshot%202024-09-25%20185509.png)

10. Update your application's configuration with the following database information

---

### Create an Docker Image from Container

1. Open your application and create a file named Dockerfile

It should be on the same level as your  “src” folder, settings.gradle and etc!

Example here: 

![Screenshot 2024-09-25 165613.png](./images/Screenshot%202024-09-25%20165613.png)

2. Add the following to the Dockerfile

```docker
# Build Stage
FROM eclipse-temurin:17-jdk-alpine as builder

WORKDIR /home/gradle/src
COPY . .
RUN ./gradlew build --no-daemon

# Runtime Stage
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app
COPY --from=builder /home/gradle/src/build/libs/DeployAWS*.jar app.jar # In 
EXPOSE 8080 
CMD ["java", "-jar", "app.jar"]
```

Do the following : 

- On the `COPY --from=builder` line, replace "DeployAWS" with your project folder name.
- If you changed which port your application loads up on, change it back to the default 8080 (Check application.properties).

### Now, before you continue, run your application once to ensure everything is working correctly!

3. Let's start by using the cd command to navigate to the correct folder. When you enter an ls command, it should look something like this

![Screenshot 2024-09-25 191504.png](./images/Screenshot%202024-09-25%20191504.png)

4. Now that your in the correct directory, check for your project

```powershell
gcloud projects list
```

If no projects show up, then just  run the below to create one, then attempt the command again

```powershell
gcloud projects create LaunchCodeCapstonePersonal --name="LaunchCode Capstone"
```

You can replace "LaunchCodeCapstonePersonal" and "LaunchCode Capstone" with names that better suit your project.

![Screenshot 2024-09-25 191851.png](./images/Screenshot%202024-09-25%20191851.png)

- **Copy and Note down your Project_id !**
5. Build Docker Image

gcr.io → is mandatory, not changeable

launchcodecapstonezachary → Your Selected Project ID

testapplication → could be anything surrounding your application

```powershell
docker build -t gcr.io/launchcodecapstonezachary/testapplication
```

6. Push Your docker image to GCR, so google has access

```powershell
docker pushgcr.io/launchcodecapstonezachary/testappplication
```

Successfully added looks like this : 

![Screenshot 2024-09-25 193920.png](./images/Screenshot%202024-09-25%20193920.png)

You can also confirm the successful upload by visiting https://console.cloud.google.com/artifacts?referrer=search&hl=en&. Make sure you've selected the correct project and look for "gcr.io": 

![Screenshot 2024-09-25 194053.png](./images/Screenshot%202024-09-25%20194053.png)

![Screenshot 2024-09-25 194314.png](./images/Screenshot%202024-09-25%20194314.png)

---

### Upload into Cloud Run

1. In the Google Cloud Console search bar, type "Cloud Run" and select the first option that appears.

![Screenshot 2024-09-25 194530.png](./images/Screenshot%202024-09-25%20194530.png)

2. Click deploy container and choose Service

![Screenshot 2024-09-25 194711.png](./images/Screenshot%202024-09-25%20194711.png)

3. Set the correct information for the Cloud Run deployment:
    - Select the container image URL by clicking the blue "Select" button

![Screenshot 2024-09-25 194916.png](./images/Screenshot%202024-09-25%20194916.png)

Select the container you uploaded

![Screenshot 2024-09-25 194858.png](./images//Screenshot%202024-09-25%20194858.png)

I selected the following for the other options: 

![Screenshot 2024-09-25 195310.png](./images/Screenshot%202024-09-25%20195310.png)

Also for those, who use things like environment variables, they allow you add those in the section below the current!

4. Click “Create”, it should then take a minute to create but it eventually should give you an url, which runs your springboot api

![Screenshot 2024-09-25 195727.png](./images//Screenshot%202024-09-25%20195727.png)

### More Information

What we currently have is a container deployed on GCP Cloud Run, commonly known as serverless, which makes it highly cost-effective. This affordability stems from the fact that your application/API spins up only when someone calls the URL provided to you. This results in a slightly slower speed during what's called a "cold start" — when your application hasn't been used for a while, it takes a bit longer to run the next time it's accessed. It's not a major issue, typically taking less than five seconds, and all subsequent API calls perform at the standard speed of a regular application. This approach allows the cloud provider to avoid dedicating a server to your application full-time, which we leverage for the free tier! Google, in particular, offers the Cloud Run service that's highly compatible with Spring Boot and has a free tier, unlike AWS Fargate.

However, we've implemented some potentially risky security practices, primarily by allowing our database to accept connections from many IP addresses. This means that if anyone obtains our database information, they could potentially make changes to our database, including deleting schemas. To mitigate this risk, you could explore ways to make your database private, such as using Virtual Private Clouds (VPCs). However, I found this approach to be even more complex.
