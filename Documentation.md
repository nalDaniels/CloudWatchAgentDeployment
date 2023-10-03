# Purpose: To set up a VPC and use a nginx load balancer to route traffic to our application. Jenkins was used to run the CI/CD pipeline as well as send an email notification to alert me that the build completed. CloudWatch was used to monitor the health and resources used by the application. I also learned to change the instance type in cases where you are using more  or less CPU and memory to ensure the virtual machine doesn’t get overwhelmed or underutilized. 

## Configure VPC Infrastructure

### Purpose: To create a virtual network so that our infrastructure has access to each other and the internet. Also, set up two availability zones in case one goes down, traffic can be routed to the other zone. 

<img width="1081" alt="Screen Shot 2023-09-30 at 2 26 20 PM" src="https://github.com/nalDaniels/Deployment4/assets/135375665/45ba1704-cfff-4b06-8efe-be3d3165f0e8">


### Steps:

1. Create the VPC, subnets, availability zones, route tables, and internet gateway using the VPC and more option


## Set Up EC2 Instance

### Purpose: To create a Jenkins server to run our CI/CD pipeline, a a nginx server to route traffic to our application, and a CloudWatch agent to monitor our application. 

### Steps:

1. Create a security group that has ports 80, 8080, 8000, and 22 open
2. Set up a t2.medium instance in the public subnet A and auto-assign it a public IP address
3. Install Jenkins, python3-pip, and nginx
4. Edit the nginx default file - this set up the location of server to the IPAddress on port 8000 and it listens from port 5000
    1. It seems we configured nginx as a load balancer which exists on port 5000 to route traffic to the IP address on port 8000
5.  Create an IAM role for CloudWatch to have permission of cloudwatchagentadminpolicy to observe and monitor infrastructure and create  and update logs
6. Modify IAM role on EC2
7. Install cloudwatch via the command line

### Optimization:

To optimize this step, I could create a script to install CloudWatch instead of manually entering each command. 

Also, I could explore how to  install the Pipeline Keep Running plugin via the command line. 

## Create Repository

### Purpose: Create a place to host the application code and files. This is useful to track changes to the branch and push files into Jenkins

### Steps: 

1. Use a terminal to clone repository
2. Adjust the URL of the  remote origin to the new repository
3. Add the username and email of GitHub
4. Create a second branch and Edit the Jenkinsfile. Add and commit those changes
    1. git branch second
    2. git switch second
    3. update file
    4. git add . and git commit -m “Added clean and deploy stages to Jenkinsfile”
5. Merge those changes into the main branch
    1. git switch main
    2. git merge second
6. Git push cloned repository to new repository
7. Optimized this process by adding a webhook

## Configure an alert on CloudWatch

### Purpose: To create alerts when the resources exceed a certain threshold

### Steps:
1. Create an alarm for when cpu_usage_user maximum goes over 21%
2. Create an alarm for when memory goes over 70%

<img width="1059" alt="Screen Shot 2023-10-02 at 11 01 51 PM" src="https://github.com/nalDaniels/Deployment4/assets/135375665/442ec379-f99d-4f6d-8110-b96a3ac6498e">


### Why these metrics?

I chose to monitor memory because I was running multiple services on one instance such as Jenkins, CloudWatch, and the nginx web server hosting the application. I also chose to monitor CPU because we are running Jenkins builds as well as the resources needed to shorten the URLs.

### Optimization

In the case that, CPU goes over a certain percentage, create another instance with higher CPU.

## Create an email notification on jenkins

### Purpose: To send an alert once the build has been completed

### Issue: I was unable to locate the post-build actions on the multibranch pipeline, so I had to add the stage/step to the jenkins file.

### Steps

1. Ensure that the email extension plugin is installed 
2. Go to manage jenkins > system > email notification 
3. add smtp.gmail.com as server and create an app password for Jenkins in your google account and input port 465
4. same thing for extended email notification then select default triggers
5. I used the Jenkins pipeline syntax tool to create a email notification
6. Updated the Jenkins file using git and reran the build
<img width="824" alt="Screen Shot 2023-10-02 at 10 57 38 PM" src="https://github.com/nalDaniels/Deployment4/assets/135375665/9fce2223-cd00-4d04-9a3f-80524b497e80">


## Create a Jenkins Pipeline

### Purpose: To automate the CI/CD pipeline 

### Steps:

1. Install Pipeline keep Running plugin
2. Create pipeline to pull repository from github
3. Run build

### Console Output and ChatGPT

The ‘build’ stage is creating a virtual environmentJenkins used git to clone the repository and fetch the latest commits. The build stage created a virtual environment using python3, activated that test environment, installed python's package manager and all of the packages in the requirements.txt that are necessary for the application to run.  The test stage activated the test environment, executed tests, and saved the results in test-reports/results.xml

I ran the commands in the ‘Clean’ stage and found that the script searched the processes for gunicorn, a HTTP server and if the PID was a number other than 0, meaning the process was active, then it killed the process. Plainly, it kills any running gunicorn processes. However, when I ran the command there was more than one gunicorn process, so this script would only rid of one using the head -n 1 command. Though, maybe killing one process, kills them all. 

The deploy stage used the ‘keep running’ to keep the deploy stage running even after the build is completed. The script downloaded the required files necessary to run the application. Then, I prompted chatGPT to act as a bash interpreter to help explain what this code meant  `python3 -m gunicorn -w 4 application:app -b 0.0.0.0 --daemon.`

I learned that it ran the gunicorn python module and created 4 agents to manage the incoming requests. This explains why I saw 4 gunicorn processes. It specified the application.py file to serve as the app file and made the application reachable outside of the instance. The ‘daemon’ option ensured that the processes ran in the background.

### Email notification
<img width="527" alt="email notification" src="https://github.com/nalDaniels/Deployment4/assets/135375665/1f9c23b3-fecf-4acc-b7d8-35262f8ea581">

### Successful Deployment
<img width="1255" alt="url deployed" src="https://github.com/nalDaniels/Deployment4/assets/135375665/558113bd-2d9c-423c-9e18-4eacd1db50b5">



## Monitor Application

### Purpose: To monitor resources used by our EC2 instance and application and determine if the instance is managing the services well

### Steps:

1. Monitor CPU and memory when running another build

### Observation:
1. How is the server performing? Can the server handle everything installed on it? if yes, how would a T.2 micro handle in this deployment? 

The t2.medium is performing well. The CPU max and Mem max haven’t exceeded 50%. The t2.micro has half the memory and CPU, so it possible that the t2.micro’s memory could be overwhelmed when running all the services concurrently, since the t2.medium was almost at 50%. I would assume the performance of the services would decline. 

2. What happens to the CPU when you run another build?
The CPU increases when running another build - memory also increased. I assume this is because we are running multiple services at the same time.

<img width="1003" alt="Screen Shot 2023-10-02 at 11 00 37 PM" src="https://github.com/nalDaniels/Deployment4/assets/135375665/90d1736c-db1c-42f6-9c18-4ba35572ebd8">


# Resources:
Please find my system design for this deployment here:
