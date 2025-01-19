# Serverless Web App - RemixKing.net

Russell here again. This time I'll show the steps I'm taking to create and secure my Serverless AWS Web App.

The end result is to get my web app allowing only authenticated users access to upload music files (only) to my backend data bucket.
The authentication is done through Cognito, and the app utilizes Api-Gateway, IAM, Lambda, HTML/Javascript and S3 Buckets services also to achieve the working end result.

I will do different versions of this project, as it's a work in progress. 
In this initial setup the JWT token will be exposed (unsafely) in the URL. I know its unsafe but I want to run through all scenarios to develop my experience with building and fixing different issues.

##

PART 1: AWS CONSOLE SETUP:

![management logo](https://github.com/user-attachments/assets/a98c57d2-686a-4e41-a2eb-cddb53a8ac3f)
##
First create an EC2 instance in AWS using the console, using a pre made VPC. In this VPC make I made sure to add an Internet Gateway and configure the Route Tables to allow connectivity to the internet for the EC2 in the public subnet. Also configure the NACLS and SGs to allow internet access on HTTP (80) and HTTPS (443).

![3  ec2 SC rules](https://github.com/user-attachments/assets/c46091b1-6f93-44f5-b19e-29fa7013ae20)

I leave these ports open to the public since I am using this instance as a web server.
