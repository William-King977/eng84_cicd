# Continuous Integration/Continuous Deployment with Jenkins
## Software Development Life Cycle (SDLC)
### Stages
* Plan/Design
* Develop
* Test
* Deploy

### Problems with traditional SDLC
* Scale and complexity
  * Functions
  * Architecture
  * Infrastructure
  * Developers
  * Operations
* Manual and slow process
* Broken communication
* Human errors
* Large volume of testing
* Long deployment window
* High cost

### DevOps in SDLC
* DevOps came to help modern SDLC - Agile
  * Collaboration
  * Automation
* DevOps Lifecycle:
  ![image](https://user-images.githubusercontent.com/44005332/116782806-2e59b200-aa83-11eb-8df2-a8a952259b07.png)


## CI/CD
### Continuous Integration (CI)
Practice of merging all developers' working copies to a shared mainline several times a day.
* Development
* Testing
* Integration

### Continuous Delivery (CDE)
Where teams produce software in short cycles, ensuring that the software can be reliably released at any time and, when releasing the software, doing so **manually**.

### Continuous Deployment (CD)
Same as CDE, but deployment is **automated**.

### Pipeline phases
1. Commit 
2. Build
3. Automate tests
4. Deploy

### Why CI/CD matters
* Reduce cost
* Faster release rate
* Smaller code changes
* Fault isolations
* More test reliability
* Increase team transparency and accountability
* Easy maintenance and updates

# Creating a Jenkins CI/CD pipeline
## Step 1: Generate a Public/Private key pair
1. Enter `ssh-keygen -t rsa -b 4096 -C "the_email_address_you_signed_up_to_github_with"`, then press Enter
2. Enter a suitable name for the key pair
3. Press Enter until the key's randomart image displays
4. Go to the `~/.ssh` directory
5. Enter `cat key_name.pub` and copy the key contents
6. On the GitHub repository, go on **Settings**
7. On the side tabs, click on **Deploy keys** > **Add deploy key**
8. Enter a suitable title and paste the key
9. Ensure **Allow write access** is checked and click **Add key**

## Step 2: Create a Webhook
1. On your repository, click on **Setting**
2. On the side tabs, click on **Webhooks** > **Add webhook**
3. Enter the Payload URL as `http://jenkins_ip:8080/github-webhook/`
4. For the Content type, select `apllication/json`
5. For the events to trigger, select **Send me everything**
6. Ensure **Active** is checked and click **Add webhook**

## Step 3: Continuous Integration Build
### General section
1. Check `Discard old builds` and keep the max number of build to 2
2. Check `GitHub project` and add the HTTP URL of the repository

### Office 365 Connector section
* Check `Restrict where this project can be run`, then set it as `sparta-ubuntu-node`

## Step 4: Merge Build

## Step 5: EC2 Instance for Deployment
We will deploy our application on an EC2 instance.
1. Create a new EC2 instance
2. Choose `Ubuntu Server 16.04 LTS (HVM), SSD Volume Type` as the AMI
3. Choose `t2.micro` as the instance type (the default)
4. On Configure Instance Details:
   * Change the VPC to your VPC
   * Change subnet to your public subnet
   * Enable `Auto-assign Public IP`
5. Add a tag with the `Key` as `Name` and enter an appropriate name as the value
6. Enter a suitable name and description for the Security Group with the following rules:
   * SSH (22) with source `My IP` - allows you to SSH
   * SSH (22) with source `jenkins_server_ip/32` - allows Jenkins server to SSH
   * HTTP (80) with source `Anywhere` - allow access to the app
7. Review and Launch
8. Select the existing DevOpsStudent key:pair option for SSH
9. Ensure the public NACL allows SSH (22) with source `jenkins_server_ip/32`

## Step 6: Continuous Deployment Build

## Bonus Step: Running in Vagrant
1. Run both the app and database using `vagrant up app` and `vagrant up db` respectively on separate terminals
2. SSH into the app using `vagrant ssh app`
3. Navigate to the `/home/ubuntu/` directory (you are placed in `/home/vagrant/` by default)
4. Run `node seed.js` in the `seeds` directory to populate the database
5. On a web browser, enter `development.local` to show that the app is *working*
6. The posts page will open with `development.local:3000/posts`
