Booking Appointment Application

A lightweight web application for handling appointment bookings. Users can submit a booking form hosted on an EC2 instance, and the backend stores booking data in an S3 bucket while sending notification emails via Amazon SES.
Tools Used

    Amazon EC2 – for running the backend and frontend
    NGINX – as the web server and reverse proxy
    Node.js + Express – for backend logic
    Amazon S3 – to store booking data
    Amazon SES – to send booking confirmation/notification emails

Step-by-Step Setup Guide
1. Create the S3 Bucket

    Go to AWS S3 and create a new bucket (e.g., my-booking-app-bucket)
    No special configuration needed

2. Set Up Amazon SES

    Navigate to Amazon SES → Verified Identities
    Add and verify an email address to use as the sender
    Stay in SES Sandbox mode (no need to move to production for this test)
    Also verify your recipient email if you're still in sandbox mode

3. Launch the EC2 Instance

    Use Amazon Linux 2 AMI
    Configure the security group to allow:
        Port 22 (SSH)
        Port 80 (HTTP)
    Attach an IAM role with the following policies:
        AmazonS3FullAccess
        AmazonSESFullAccess

4. Connect to the EC2 Instance

ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>

5. Install Required Software

sudo yum update -y

# Install Node.js 18
curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

# Install nginx
sudo amazon-linux-extras enable nginx1
sudo yum install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx

# Install git (optional)
sudo yum install -y git

6. Set Up Project Directory

mkdir ~/booking-app && cd ~/booking-app

7. Create Project Files

    index.html – contains the frontend booking form
    server.js – backend Node.js server (uses AWS SDK v3)

You’ll need to create and edit these files with your form and server logic.
8. Initialize Node Project & Install Dependencies

npm init -y
npm install express body-parser @aws-sdk/client-s3 @aws-sdk/client-ses cors

9. Run the Node.js Server

Choose one of the following methods:
Option 1: Foreground (for development)

sudo node server.js

Option 2: Background using nohup

sudo nohup node server.js > out.log 2>&1 &

Option 3: Use PM2 (recommended for production)

sudo npm install -g pm2
pm2 start server.js
pm2 save
pm2 startup

10. Configure NGINX as a Reverse Proxy

sudo nano /etc/nginx/nginx.conf

Replace the entire content with:

events {}

http {
    server {
        listen 80;
        server_name _;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }

        location /book {
            proxy_pass http://localhost:3000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
}

Then validate and restart:

sudo nginx -t
sudo systemctl restart nginx

11. Serve the Frontend from NGINX

sudo cp index.html /usr/share/nginx/html/index.html

12. Start Your App

If not already running, start your backend server using one of the methods from step 9.
13. Access the Web App

Open your browser and go to:

http://<EC2_PUBLIC_IP>/

Submit the booking form to:

    Store booking data in your S3 bucket
    Send a notification email to the configured recipient

Notes

    SES in sandbox mode only allows emails between verified identities
    Once working, request production SES access to send to any email
    Use HTTPS and environment variables for production deployments and better security.
    Consider using pm2 to auto-start your app on reboot

Final Project Structure

booking-app/
├── index.html
├── server.js
├── package.json
└── node_modules/

Sample Email Output (via SES)

Subject: New Booking Notification

New booking received:
Name: John Doe
Email: john@example.com
Date: 2025-07-01
Time: 10:30 AM

