# MarketPeak Ecommerce Website

This project downloads a free e-commerce website for a new marketplace named "MarketPeak" from https://www.tooplate.com/view/2082-pure-mix. Using Git as a version control the platform will be developed on a Linux environment and deployed on an AWS EC2 instance.

## 1. Implement Version Control with Git

**1.1 Initialise Git Repository:**

Create MarketPeak Ecommerce Folder and initialise Git.

![Create Folder](images\readme-img.png)

**1.2 Download the template:**

Download the specific template of choice and customize.

![Download MarketPeak Template](images\readme-img1.png)

**1.3 Stage and Commit the Template to GitHub:**

Stage and commit the files to GitHub repository.

![Stage and Commit](images\readme-img2.png)

## 2. Aws Deployment:

Setup an Amazon EC2 instance to deploy MarketPeak_Ecommerce platform.

**2.1 Set Up an AWS EC2 instance**

* Log in to AWS Management Console, Launch an EC2 instance using Amazon Linux AMI, and Connect to the instance using SSH.

![EC2 Instance](images\readme-img3.png)

**2.2 Clone the Repository on the Linux Server**

* Install git, clone the Repository to the EC2 instance using HTTPS or SSH.

![Clone Repository](images\readme-img4.png)

**2.3 Configure httpd for Website**

* Clear the default httpd web directory, copy the MarketPeak Ecommerce files to it and reload the httpd service.

![Configure Website](images\readme-img5.png)

**2.4 Access Website from Browser**

* Open a web browser and access the public IP of your EC2 instance to view the deployed website.

![MarketPeak Website](images\readme-img6.png)

## Continuous  Integration and Deployment Workflow

This covers how changes are made in a development environment, Using Git version control and the deployment of updates to your production server on AWS.

**3.1 Developing New Features and Fixes**

* Create development branch and implement changes/fix bugs on your website. 

* Push development branch to GitHub, Create Pull Request, Review and Merge.

![Merge Development Branch](images\readme-img7.png)

**3.2 Deploying Updates to the Production Server**

* SSH into your AWS EC2 instance where the production server is hosted. Navigate to the website's directory and pull the latest changes fro the main branch. Reload the server.

![Pull Update](images\readme-img8.png)

**3.3 Updated MarketPeak Ecommerce Website**

* Access the website to confirm if the changes/fixes where effected.

![Updated Website](images\readme-img9.png)
