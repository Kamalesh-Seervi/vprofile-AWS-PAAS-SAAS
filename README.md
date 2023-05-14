# Vprofile-AWS-PAAS/SAAS
We will be using RDS instances, elastic cache, DB initialisation, Amazon MQ, Beanstalk, and Cloudfront

## Services:

- EC2 instances 
- ELB
- Autoscaling 
- EFS/S3
- RDS
- Elastic cache
- Active MQ
- Route 53
- CloudFront
- CloudWatch

## Work Flow:

- Create KeyPair for beanstalk instance.
- Create Security group for elastic Cache, RDS and Active MQ.
- Create RDS, Elastic cache and Active MQ.
- Create Elastic Beanstalk environment.
- Update sg of backend to allow traffic from Bean sg.
- Update sg of backend to allow internal traffic. 
- Launch Ec2-Instance for DB Initializing.
- Login to the instance and Inititialize RDS DB.
- Change healthcheck on beanstalk to /login.
- Add 443 https Listner to ELB.
- Build Artifact with Backend Information.
- Deploy Artifact to Beanstalk.
- Create CDN with ssl cert.
- Update Entry in GoDaddy DNS Zones.
- Test the URL.

## Create KeyPair for beanstalk instance.

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/7c1c4a1b-aac2-425a-8e76-91f50a33f150">

## Create Security group for elastic Cache, RDS and Active MQ.

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/a5789d4a-bacb-4bbd-bc85-703a1fd4ddef">

## Create RDS, Elastic cache and Active MQ.

- First create RDS subnet group and parameter group:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/a9fa2cc6-54ce-43cd-9247-23128ab0d265">

- Create the RDS database now:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/75d1285c-9da8-48fa-b611-117396370843">

- Create Elastic cache parameter group and subnet group:

#### Parameter Group: 

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/65dbfcc3-c285-4a88-807d-8dec2c2d1a55">

#### Subnet Group:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/c612f0ed-5ea0-45d6-aacf-a25ed4d8f6ea">

- Create elastic cache now:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/166ea457-e7bc-429b-838e-64e0b5dedadf">

- Create Rabbit MQ service:
     
<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/763946b9-316c-457a-87b9-913dee405182">

## Initialise DB with an EC2 instance:

- In userdata type the below script:

```
#!/bin/bash
sudo apt update
sudo apt install mysql-client -y
```

- Now login to the instance .
- For mac we need to allow read/write permission 
```
chmod 600 <privatekey.pem file>
```

- Now login using ssh
```
ssh -i <privatekey> ubuntu@<public ip>
```
- Allow mysql-client to 3306 in backend so change the sg of backend-sg

<img width="682" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/6fa31618-f12c-42ee-8671-085c4f3edf37">

- Now clone the below repo:

```
git clone https://github.com/devopshydclub/vprofile-project.git
```
```
cd vprofile-project
```

- Now check for aws-refactor branch

```
git branch -a
```
```
git checkout /remotes/origin/aws-refactor
```
- Go to src/main/resources then type the following command;

```
mysql -h vprofile-rds-mysql.cemfjrswo6eb.us-east-1.rds.amazonaws.com -u <username> -p<password> accounts < db_backup.sql
```
```
mysql -h vprofile-rds-mysql.cemfjrswo6eb.us-east-1.rds.amazonaws.com -u admin -p5dmatbrqpAcmzu0l0m8S accounts
```

- show tables;

<img width="682" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/bb045670-4a5c-4d76-ba33-f7736d9e4bdf">

## Create Elastic Beanstalk environment.

- IAM roles required for beanstalk:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/db580a02-b4e4-4cbf-a4d5-10b6dd98f15f">

- Service roles

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/b0f9ffbd-fc09-41b5-bc37-a47a0170f5c0">

- Image created for beanstalk:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/7af22f27-1514-4a0f-826c-5c036e107fe3">

- Check by pasting the domain :

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/73237202-7897-49db-8087-e487c080aa67">

## Update sg of backend to allow traffic from Bean sg:

- Go to ownership in s3 bucket:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/167d3440-cbc9-44d7-a125-ecb6fca54c8f">

- Then in elastic beanstalk go to configuration in that add process:

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/e90ccba0-dc6f-4932-8b1b-9ee540963e03">

- Take the sg id of the app paste in backend-sg and add all traffic to it:(so that the EC@ instance can communicate with backend services)

<img width="1552" alt="image" src="https://github.com/Kamalesh-Seervi/vprofile-AWS-PAAS-SAAS/assets/107933310/aa614291-d75d-41dc-8ad8-4dcfe24d273b">

