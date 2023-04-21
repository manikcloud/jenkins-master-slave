# Setting Up Jenkins Slave Node on AWS EC2 Instance
To spin up a Jenkins slave node in AWS, you can use EC2 instances. Here's a step-by-step guide to setting up a Jenkins slave node on an EC2 instance:


## Prerequisites:

- An AWS account
- A running Jenkins master
- AWS CLI and EC2 key pair configured on your local machine or Jenkins master

### Steps:

1. Create a security group: In the AWS Management Console, go to the EC2 dashboard and click on "Security Groups" in the left-hand menu. Click "Create security group" and configure it with the necessary inbound rules (e.g., SSH, and any other required ports).

2. Create an IAM role: Go to the IAM dashboard and create a new role for your EC2 instance. Attach appropriate policies to the role, such as AmazonEC2FullAccess or any custom policies you need.

3. Launch an EC2 instance: In the EC2 dashboard, click on "Instances" in the left-hand menu and click "Launch Instance". Choose an Amazon Linux 2 AMI (or your preferred OS) and an appropriate instance type. In the "Configure Instance Details" step, select the IAM role you created earlier. In the "Add Storage" step, add any additional storage needed for your Jenkins workspace. In the "Configure Security Group" step, select the security group you created earlier. Finally, launch the instance with your existing key pair.

4. Configure the instance: SSH into the new EC2 instance using the key pair and public IP address. Install any required software (e.g., Java, build tools, etc.) and make any necessary configurations.

5. Install the Jenkins agent: Download the Jenkins agent JAR file from your Jenkins master instance and place it in a suitable location on the EC2 instance (e.g., /home/ec2-user/jenkins). You can download the JAR file using the following command:

```
wget https://[your-jenkins-url]/jnlpJars/agent.jar -O /home/ec2-user/jenkins/agent.jar

```

Replace `[your-jenkins-url]` with the URL of your Jenkins master.

6. Create a systemd service: On the EC2 instance, create a new systemd service file at `/etc/systemd/system/jenkins-slave.service` with the following content:

```
[Unit]
Description=Jenkins Slave
After=network.target

[Service]
User=ec2-user
ExecStart=/usr/bin/java -jar /home/ec2-user/jenkins/agent.jar -jnlpUrl https://[your-jenkins-url]/computer/[slave-node-name]/slave-agent.jnlp -secret [your-agent-secret] -workDir "/home/ec2-user/jenkins/workspace"
Restart=always

[Install]
WantedBy=multi-user.target
```

Replace `[your-jenkins-url]`, `[slave-node-name]`, and `[your-agent-secret]` with the appropriate values from your Jenkins master instance.

7. Start and enable the Jenkins agent: Run the following commands on the EC2 instance to start and enable the Jenkins agent service:
```
sudo systemctl daemon-reload
sudo systemctl start jenkins-slave
sudo systemctl enable jenkins-slave
```

8. Add the EC2 instance as a slave node in Jenkins: Follow the steps mentioned in the previous answer to create and configure a new Jenkins slave node. For the "Launch method", choose "Launch agent via Java Web Start" and copy the agent secret from the systemd service file.

Your Jenkins slave node on an AWS EC2 instance should now be up and running