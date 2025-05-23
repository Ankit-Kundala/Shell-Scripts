# Problem Statement :
Client was using ELK stack to store all the Logs files of the Application as in Application Microservices log(100s of services running), Kubernetes control plane logs, and Infrastructure Log files. Infrastructed logs consisted mostly of Jenkins and this Jenkins had most log files comming for the CI/CD pipeline commits( approx 500 commits/perday) that too from the lower UAT environments and Staging environment and they were other enviroments like Pre-Prod and Production. This UAT and Staging Environment logs , their Build failures and other errors were sent through gmail and Slack and get fixed too and the team would rarely do log analysis in this Jenkins log-files. They were holding storing this log-files in ELK stack just as a Backup. As ELK here is not so needed as the failures would already sent through gmail and slack and would get fixed.

# Solution :
So, the Focus was to shift the UAT and Staging envirnoment logs to S3 Buckets to reduce the cost of ELK stack . For more cost-reduction Log files can be stored in AWS Glacier and Deep archieve.

At the end of the day,the Shell script will run manually or automatically through cron jobs and trigger the Jenkins, this will pull all the Jenkins log-files and store in S3 Buckets. If we have number Pipelines running this Shell script would loop in to each Pipeline and collect data and store in S3 Buckets.

As this would result in approx 50% cost-reduction by Simply using the Below Shell Script.

## Step 1 : Install AWS CLI and Configure it
Connect the EC2 to the terminal and follow the Steps Accordingly.

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
```
aws cli
```
To check wheter the aws cli has been installed, if yes then you’ll get all the commands of cli and its successfully installed.


```
aws configure
``` 
Here you’ll be asked for AWS Access Key ID, AWS Secret Access Key and Region. You can Create a new user for the particular project or just can continue with Access Token. IAM > Dashboard > Create access Token and Connect

## Step 2 : Install Jenkins and Java

```
sudo apt update
sudo apt install openjdk-17-jre
```
Verify Java is Installed by running the following command

```
java -version
```
```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
After running this Jenkins will be installed on the system. Open the port 8080 from the Security Inbound Rules and run the [ public-ip-address/8080 ] you’ll get to see Jenkins Log in page.

## Step 3 : Write Script

![Screenshot 2025-05-10 140632](https://github.com/user-attachments/assets/ecdc9ba9-2749-4339-955a-db60f9afc1c7)

Create a new [optimization.sh] file and Copy the script , make sure Replace with your S3 bucket name

After this give needed permission to the [optimizationn.sh] .

```
chmod 777 optimization.sh
```
## Step 4 : Run CI/CD Pipeline and Create Log-file
For now I have used a dummy pipeline for testing purpose .


```
pipeline {
    agent any
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 1, unit: 'SECONDS')
    }
    stages {
        stage('Stage 1') {
            steps {
                echo 'Hello World !!!!'
            }
        }
        stage('Stage 2') {
            steps {
                echo 'Script Running !'
            }
        }
    }
}
```

Run this pipe and log-file would be created.


## Step 5 : Create a New Bucket in AWS
Amazon S3 > Buckets > Create bucket \> Jenkins-Falcon
![Screenshot 2025-05-08 231420](https://github.com/user-attachments/assets/591f47b7-e127-42c6-ab0c-183b618eeb26)
  
##Step 5 : Run this Script
```
./optimazation.sh
```
After running the script you’ll get to see files uploaded to bucket. Open AWS S3 Buckets and Check you’ll find the log-files

![Screenshot 2025-05-08 230939](https://github.com/user-attachments/assets/be68edb4-08f6-448d-88fb-f4f6ccdc77c5)


We’ve successfully transferred the Log-files to AWS we can now store this files in the S3 Glacier Deep Archive

Reference :
All thanks to Abhishek Veermala for this Project !

Project Link - https://www.youtube.com/watch?v=bR6AbqZK4LQ
