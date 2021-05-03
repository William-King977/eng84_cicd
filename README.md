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

## Step 3: Creating Jenkins Jobs
1. On the Jenkins Dashboard, click on **New Item** (side tabs)
2. Enter a suitable name for the job
3. Select **Freestyle project**
4. Click **Ok**
5. Create a job for CI, merging and deployment

## Step 4: Continuous Integration (CI) Job
### General
1. Click **Discard old builds** and keep the max number of build to 2
2. Click **GitHub project** and add the HTTP URL of the repository

### Office 365 Connector
* Click **Restrict where this project can be run**, then set it as `sparta-ubuntu-node`

### Source Code Management
1. Select **Git**
2. In **Repositories**:
   * **Repository URL:** insert the SSH URL from GitHub
   * **Credentials:** 
     * Next to **Credentials**, click **Add** > **Jenkins** 
     * Select **Kind** as **SSH Username with private key**
     * Set a suitable description and enter the private key directly. The private key is in your `~/.ssh` directory, named `key_name` without `.pub`. Ensure that the begin and end text of the key is included.
     * With the *credential* added, select the one you created
   * **Branches to build:** set to `*/dev` (dev branch)

### Build Triggers
* Click **GitHub hook trigger for GITScm polling**

### Build Environment
* Click **Provide Node & npm bin/ folder to PATH**

### Build
1. Click **Add build step** > **Execute Shell**
2. In command, enter the following code:
  ```
  cd app
  npm install
  npm test
  ```

### Post-build Actions
1. Select **Add post-build action** > **Build other projects**
2. Insert the project name for the merge job
3. Ensure **Trigger only if build is stable** is selected

## Step 5: Merge Job
### General
1. Click **Discard old builds** and keep the max number of build to 2
2. Click **GitHub project** and add the HTTP URL of the repository

### Office 365 Connector
* Click **Restrict where this project can be run**, then set it as `sparta-ubuntu-node`

### Source Code Management
1. Select **Git**
2. In **Repositories:**
   * **Repository URL:** insert the SSH URL
   * **Credentials:** select the credential you created earlier
   * **Branches to build:** set to `*/dev` (dev branch)

### Build Environment
* Select **Provide Node & npm bin/ folder to PATH**

### Post-build Actions
1. First, select **Add post-build action** > **Git Publisher**
2. Click **Push Only If Build Succeeds**
3. In **Branches**:
   * **Branch to push:** main
   * **Target remote name:** origin
4. Next, select **Add post-build action** > **Build other projects**
5. Insert the project name for the deploy job
6. Ensure **Trigger only if build is stable** is selected
7. Ensure the **Build other projects** block is below the **Git Publisher** block

## Step 6: EC2 Instance for Deployment
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
   * Custom TCP (3000) with source `0.0.0.0/0` - allow access to port 3000
7. Review and Launch
8. Select the existing DevOpsStudent key:pair option for SSH
9. Transfer the app's folders using `scp -i ~/.ssh/DevOpsStudent.pem -r app_location ubuntu@app_ec2_public_ip:~/app/` in the directory before the app
10. SSH inside the instance and run the app's provision file
11. Ensure the public NACL allows SSH (22) with source `jenkins_server_ip/32`
12. If the Jenkins server updates, the GitHub webhook, security group and NACL need to be modified

## Step 7: Continuous Deployment Job
### General
1. Click **Discard old builds** and keep the max number of build to 2
2. Click **GitHub project** and add the HTTP URL of the repository

### Office 365 Connector
* Click **Restrict where this project can be run**, then set it as `sparta-ubuntu-node`

### Source Code Management
* Keep it at **None**

### Build Environment
1. Select **Provide Node & npm bin/ folder to PATH**
2. Select **SSH Agent**:
   * Select **Specific credentials**
   * Select the SSH key for the EC2 instance (DevOpsStudent in this case)

### Build
1. Select **Add build step** > **Execute Shell**
2. In command, insert the following code:
   ```
   rm -rf eng84_cicd_jenkins*
   git clone -b main https://github.com/William-King977/eng84_cicd_jenkins.git
   cd eng84_cicd_jenkins

   rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@deploy_public_ip:/home/ubuntu/app
   rsync -avz -e "ssh -o StrictHostKeyChecking=no" environment ubuntu@deploy_public_ip:/home/ubuntu/app

   ssh -A -o "StrictHostKeyChecking=no" ubuntu@deploy_public_ip <<EOF

       killall npm

       cd app/app
       sudo npm install
       node seeds/seed.js
     
       node app.js &

   EOF
   ```
3. NOTE: the `deploy_public_ip` will need to be changed each time you re-run the deployment instance 

## Step 8: Trigger the Builds!
1. Switch to the `dev` branch
2. Make any change to your repository
3. Add, commit and push your changes to the `dev` branch
4. A CI build will trigger
5. A merge build will only trigger if the tests in the CI build pass
6. After the merge build, the `dev` branch will merge with your `main` branch on GitHub
7. If the deployment build succeeds, the app will be running on its public IP!
8. NOTE: the image will only display if you access port 3000

## Bonus Step: Running in Vagrant
1. Run both the app and database using `vagrant up app` and `vagrant up db` respectively on separate terminals
2. SSH into the app using `vagrant ssh app`
3. Navigate to the `/home/ubuntu/` directory (you are placed in `/home/vagrant/` by default)
4. Run `node seed.js` in the `seeds` directory to populate the database
5. On a web browser, enter `development.local` to show that the app is *working*
6. The posts page will open with `development.local:3000/posts`
