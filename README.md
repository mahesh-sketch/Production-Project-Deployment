# Production Deployment Guide 🚀

Welcome to the Production deployment guide ! This comprehensive guide provides step-by-step instructions for deploying your application in a production environment on AWS, incorporating key technologies like Node.js, NGINX, and Let's Encrypt. From launching an EC2 instance to configuring SSL with Let's Encrypt, each section is designed to streamline the deployment process. Additionally, the README covers best practices for production deployment, emphasizing continuous integration, monitoring, scaling, security measures, and backup strategies. Whether you're a seasoned developer or new to deployment, this README serves as your go-to resource for a smooth and successful project launch. Happy coding and deploying! 🚀🌐

## Step 1: Create an AWS EC2 Instance ☁️

1. **Sign in to AWS:**
 - Visit [AWS Console](https://aws.amazon.com/).
 - Sign in or create a new account.

2. **Navigate to EC2 Dashboard:**
 - In the AWS Management Console, click on "Services" and select "EC2" under "Compute."

3. **Launch an Instance:**
 - Click on "Instances" in the left sidebar.
 - Click the "Launch Instance" button.

4. **Choose an Amazon Machine Image (AMI):**
 - Select an Ubuntu AMI (e.g., Ubuntu Server 20.04 LTS).

5. **Choose an Instance Type:**
 - Select a t2.medium instance type (or any other preferred type).

6. **Configure Instance:**
 - Configure instance details, network settings, and IAM role if needed. Click "Next."

7. **Add Storage:**
 - Specify storage. Default settings are usually fine. Click "Next."

8. **Add Tags (Optional):**
 - Add tags for better organization (optional). Click "Next."

9. **Configure Security Group:**
 - Create  security group.
     - Allow SSH traffic from.
     - Allow HTTPS traffic from the internet.
     - Allow HTTP traffic from the internet
 - Click "Review and Launch."

10. **Review and Launch:**
 - Review configurations. Click "Launch."

11. **Create Key Pair:**
 - Choose "Create a new key pair."
 - Give it a name (e.g., YourKeyPairName).
 - Click "Download Key Pair" and keep it secure.

12. **Launch Instances:**
 - Click "Launch Instances."

13. **View Instances:**
  - Click "View Instances" to go back to the EC2 dashboard.

14. **Connect to Instance:**
 - Once the instance is "running," select it and click "Connect" for SSH instructions.

## Step 2: Allocate Elastic IP Address 🌐

1. **Navigate to Elastic IPs:**
 - In the EC2 dashboard, click on "Elastic IPs" in the left sidebar.

2. **Allocate New Address:**
 - Click on "Allocate Elastic IP address."
 - Choose "Amazon's pool" for the allocation method.
 - Click "Allocate."

3. **Associate Elastic IP:**
 - Select the newly allocated Elastic IP.
 - Click "Actions" and choose "Associate Elastic IP address."

4. **Associate with Instance:**
 - Choose the instance you created.
 - Click "Associate."

5. **View Updated Public IP:**
 - Go back to the EC2 dashboard.
 - Check your instance's details to find the updated Public IP.

## Step 3: Connect to EC2 Instance Using SSH Client on Windows 💻

1. **Open Windows Terminal:**
 - Open Windows Terminal on your local machine.

2. **Navigate to the Directory with Key Pair:**
 - Change your working directory to the location where your key pair (`YourKeyPairName.pem`) is stored.


3. **Copy SSH Command from AWS Console:**
 - In the AWS Console, navigate to your EC2 instance details.
 - Copy the SSH command provided.

4. **Paste and Run SSH Command in Windows Terminal:**
 - Paste the copied command into your Windows Terminal.

    ```bash
    ssh -i "YourKeyPairName.pem" ubuntu@your-elastic-ip 
    ```

   - Press Enter to execute the command.

5. **SSH into Your EC2 Instance:**
 - You should now be connected to your EC2 instance via SSH.

## Step 4: Install Node and NPM 🚚

1. **Download and Install Node.js:**
 - Execute the following command to download and run the NodeSource setup script, adding the Node.js repository:

    ```bash
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    ```

2. **Install Node.js and NPM:**
 - Install Node.js and NPM by running:

    ```bash
    sudo apt install nodejs
    ```

3. **Check Node.js Version:**
 - Verify the installation by checking the Node.js version:

    ```bash
    node --version
    ```
 - The displayed version confirms a successful installation.

## Step 5: Clone Your Project from GitHub 🔄

    git clone <Repositories Link>

## Step 6: Install Dependencies and Test App 📦

1. **Install PM2 globally (if not installed):**
 - Use the following command to install PM2 globally. This command checks if PM2 is already installed; if not, it installs it:

    ```bash
    sudo npm install pm2 -g
    ```

 - PM2 is a process manager for Node.js applications, ensuring they run continuously.

2. **Navigate to Your Project Directory:**
 - Change your current working directory to the directory where your project is located. Replace `short-url-nodejs` with your actual project directory:

    ```bash
    cd path/to/your/project/short-url-nodejs
    ```

3. **Start the App with PM2:**
 - Start your Node.js app using PM2. Replace `index` with the entry file of your application:

    ```bash
    pm2 start index
    ```

 - PM2 will manage your application, keeping it running in the background.

4. **PM2 Commands for Managing the App:**
 - Use the following commands to manage your app with PM2:

    ```bash
    pm2 show app
    pm2 status
    pm2 restart app
    pm2 stop app
    pm2 logs   # Show log stream
    pm2 flush  # Clear logs

    # To ensure the app starts when the system reboots
    pm2 startup ubuntu
    ```
## Step 7: Setup Firewall 🛡️

1. **Update Inbound Rules in Security Group:**
 - Go to the AWS EC2 Dashboard.
 - Click on "Security Groups" in the left sidebar.
 - Select the security group associated with your EC2 instance.

 - Edit the inbound rules and add two custom TCP rules for your application's port (replace `your-port-number`):
    - **Rule 1:** Allow incoming traffic from any IPv4 address:
    - Type: Custom TCP
    - Port Range: your-port-number
    - Source: 0.0.0.0/0

    - **Rule 2:** Allow incoming traffic from any IPv6 address:
    - Type: Custom TCP
    - Port Range: your-port-number
    - Source: ::/0

2. **Check if Your App is Running:**
- Open your web browser and enter the following URL with your EC2 instance's public IP address:

    ```
    http://your-ec2-public-ip:your-port-number
    ```

- Replace `your-ec2-public-ip` with the actual public IP address of your EC2 instance, and `your-port-number` with the port your application is running on.

- If your app is running, you should see your application's interface or a response.

## Step 8: Install NGINX and Configure 🌐

1. **Install NGINX:**
 - Run the following command to install NGINX:

    ```bash
    sudo apt-get install nginx
    ```

2. **Configure NGINX:**
 - Open the NGINX default configuration file for editing:

    ```bash
    sudo nano /etc/nginx/sites-available/default
    ```

 - Add the following configuration to the `location` part of the `server` block. Replace `yourdomain.com` and `www.yourdomain.com` with your actual domain, and update the `proxy_pass` with your app's port:

    ```nginx
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:your-app-port;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    ```

3. **Check NGINX Configuration:**
 - Run the following command to check the NGINX configuration:

    ```bash
    sudo nginx -t
    ```

   - This ensures that the NGINX configuration is correct.

4. **Restart NGINX:**
 - Restart NGINX to apply the changes:

    ```bash
    sudo nginx -s reload
    ```
## Step 9: Add SSL with Let's Encrypt 🔒

1. **Add Certbot Repository:**
 - Add the Certbot repository to your system:

    ```bash
    sudo add-apt-repository ppa:certbot/certbot
    ```

2. **Update Package List:**
 - Update the package list to include the Certbot packages:

    ```bash
    sudo apt-get update
    ```

3. **Install Certbot and NGINX Plugin:**
 - Install Certbot and the NGINX plugin for automatic certificate configuration:

    ```bash
    sudo apt-get install python3-certbot-nginx
    ```

4. **Obtain SSL Certificates:**
 - Obtain SSL certificates for your domain and its subdomains. Replace `yourdomain.com` and `www.yourdomain.com` with your actual domain:

    ```bash
    sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
    ```

5. **Test Certificate Renewal Process:**
 - Certificates from Let's Encrypt are valid for 90 days. Test the renewal process with the following command:

    ```bash
    certbot renew --dry-run
    ```

 - This ensures that the renewal process works correctly.


Your website should now be deployed and accessible securely with SSL.🌟

