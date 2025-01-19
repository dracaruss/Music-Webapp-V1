# Apache-EC2-Server-Setup

Russell here this time to show the steps I took to create my Serverless AWS Web App!

In this instance I am first configuring the app to use authentication via Cognito using the implicit method, where the token is exposed in the URL.
In subsequent builds, I'll use the 'code'

Follow me as I demonstrate how to set up an Apache web server on EC2 in AWS in 3 different ways. 
First through the AWS console.
Second by terraform.
Third by using Jenkins to automate the terraform and entire workflow.

##

PART 1: AWS CONSOLE SETUP:

![management logo](https://github.com/user-attachments/assets/a98c57d2-686a-4e41-a2eb-cddb53a8ac3f)
##
First create an EC2 instance in AWS using the console, using a pre made VPC. In this VPC make I made sure to add an Internet Gateway and configure the Route Tables to allow connectivity to the internet for the EC2 in the public subnet. Also configure the NACLS and SGs to allow internet access on HTTP (80) and HTTPS (443).

![3  ec2 SC rules](https://github.com/user-attachments/assets/c46091b1-6f93-44f5-b19e-29fa7013ae20)

I leave these ports open to the public since I am using this instance as a web server.
