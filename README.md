# Creating and configuring a slave node in Jenkins

To create and configure a slave node in Jenkins, follow these steps:

1. Install Java on the slave node: Ensure that Java is installed on the slave machine. You can download the appropriate Java version from the official website.

2. Open Jenkins: Log in to your Jenkins master instance with administrative privileges.

3. Navigate to 'Manage Jenkins': Click on "Manage Jenkins" from the left-hand menu.

4. Access 'Manage Nodes and Clouds': Click on "Manage Nodes and Clouds" under the "System Configuration" section.

5. Create a new node: Click on the "New Node" button on the left-hand side.

6. Enter node details: Provide a name for the new slave node, select "Permanent Agent" and click "OK".

7. Configure the node: Fill in the necessary details for the slave node:
  - Description: Add a brief description of the node (optional).
  - executors: Set the number of concurrent build jobs the slave node can handle.
  - Remote root directory: Specify a directory on the slave machine where Jenkins will store its files (e.g., /home/jenkins/agent).
  - Labels: Add labels to the node to help identify its purpose or capabilities (e.g., "linux", "php", etc.).
  - Usage: Choose how Jenkins should utilize this node. Select "Use this node as much as possible" to allow any jobs to run on it, or "Only build jobs with label expressions matching this node" to restrict its usage.
  - Launch method: Select a method for connecting the slave node to the Jenkins master. The most common methods are "Launch agent via Java Web Start" or "Launch agent via SSH". In this example, we'll choose "Launch agent via SSH".
  - Host: Enter the hostname or IP address of the slave machine.
  - Credentials: Add or select the appropriate SSH credentials for connecting to the slave machine.
  - Host Key Verification Strategy: Choose a strategy for verifying the host key (e.g., "Manually trusted key Verification Strategy" or "Non verifying Verification Strategy").

8. Save the configuration: Click "Save" to finalize the node configuration.

9. Connect the node: After saving, Jenkins will attempt to connect to the slave node. If successful, the node's status will change to "Connected" on the "Manage Nodes and Clouds" page.

You have now created and configured a slave node in Jenkins. The Jenkins master will distribute jobs to this node based on the configured usage and label settings.

-----------------------------------------------------------------------------------------------------------

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