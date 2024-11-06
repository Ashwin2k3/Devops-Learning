To create a CI/CD pipeline on your virtual machine that triggers builds on GitHub commits, you can set up a system using tools like **GitHub Actions** and **webhooks** or a CI/CD tool such as **Jenkins** or **GitLab CI/CD**. Here's a straightforward approach to accomplish this:

### Option 1: Using GitHub Actions (Recommended for GitHub Repos)

1. **Create a `.github/workflows` Directory**: Inside your repository, create a folder named `.github/workflows`. This is where your GitHub Actions workflows will be stored.

2. **Create a Workflow File**:
   - Inside the `.github/workflows` folder, create a YAML file, like `ci-cd.yml`.
   - Define the workflow in this file to run on `push` events. Below is a sample workflow to pull the latest changes and build the project on your virtual machine.

   ```yaml
   name: CI/CD Pipeline

   on:
     push:
       branches:
         - main  # Trigger on commits to the main branch

   jobs:
     build:
       runs-on: ubuntu-latest  # GitHub's virtual environment, change this if you want

       steps:
         - name: Checkout Code
           uses: actions/checkout@v2

         - name: Set up Node.js (example for a Node.js project, modify as needed)
           uses: actions/setup-node@v2
           with:
             node-version: '14'

         - name: Install Dependencies
           run: npm install

         - name: Run Build
           run: npm run build

         - name: Deploy to VM
           env:
             SERVER_IP: ${{ secrets.SERVER_IP }}
             SERVER_USER: ${{ secrets.SERVER_USER }}
             SERVER_SSH_KEY: ${{ secrets.SERVER_SSH_KEY }}
           run: |
             ssh -i $SERVER_SSH_KEY $SERVER_USER@$SERVER_IP "cd /path/to/app && git pull && npm install && npm run build"
   ```

   In this example:
   - **Install Dependencies** and **Run Build** steps are executed.
   - The `Deploy to VM` step uses SSH to deploy the build to your VM by pulling the latest code and running the necessary commands.
   
3. **Set Up Secrets in GitHub**:
   - Go to **Settings > Secrets** in your GitHub repository and add the following secrets:
     - `SERVER_IP`: The IP address of your VM.
     - `SERVER_USER`: The SSH username.
     - `SERVER_SSH_KEY`: Your private SSH key to authenticate to your VM.

4. **Install SSH Keys on VM**:
   - Make sure your VM has the corresponding public SSH key in the `~/.ssh/authorized_keys` file to allow GitHub Actions to SSH into the VM.

### Option 2: Using Jenkins for More Customizability

1. **Install Jenkins on Your VM**:
   - Install Jenkins on your VM and make sure it’s accessible. Jenkins can be installed on Linux-based VMs with:
     ```bash
     sudo apt update
     sudo apt install openjdk-11-jdk -y  # Java dependency
     wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
     sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
     sudo apt update
     sudo apt install jenkins -y
     sudo systemctl start jenkins
     ```

2. **Set Up Jenkins Pipeline**:
   - Log into Jenkins and create a new **Pipeline** job.
   - Add a GitHub Webhook to trigger Jenkins builds on new commits by going to **GitHub > Settings > Webhooks** and specifying the Jenkins endpoint (e.g., `http://your-vm-ip:8080/github-webhook/`).

3. **Define Your Jenkinsfile**:
   - Add a `Jenkinsfile` to your repository with steps to pull the latest code, build, and deploy to your application. Example:

     ```groovy
     pipeline {
       agent any
       stages {
         stage('Checkout') {
           steps {
             git 'https://github.com/your-username/your-repo.git'
           }
         }
         stage('Build') {
           steps {
             sh 'npm install && npm run build'
           }
         }
         stage('Deploy') {
           steps {
             sshagent(credentials: ['your-ssh-key-id']) {
               sh 'ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "cd /path/to/app && git pull && npm install && npm run build"'
             }
           }
         }
       }
     }
     ```

4. **Configure Environment Variables in Jenkins**:
   - In Jenkins, go to **Manage Jenkins > Manage Credentials** and add the necessary SSH credentials.
   - Replace `$SERVER_USER` and `$SERVER_IP` with your VM’s SSH details.

### Summary

For GitHub projects, **GitHub Actions** is a convenient choice as it directly integrates with GitHub and requires fewer dependencies on your VM. For more complex setups and flexibility, **Jenkins** provides advanced configuration options.

Choose the one that best fits your project requirements!
