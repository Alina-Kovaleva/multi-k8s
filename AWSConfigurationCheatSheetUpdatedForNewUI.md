# AWS Multi-Docker Deployment Cheat Sheet

## Elastic Beanstalk Application Creation

1. Go to AWS Management Console and use Find Services to search for Elastic Beanstalk
2. Click “Create Application”
3. Set Application Name to 'multi-docker'
4. Scroll down to Platform and select Docker
5. Verify that "Single Instance (free tier eligible)" has been selected
6. Click the "Next" button.
7. In the "Service Role" section, verify that "Use an Existing service role" is selected.
8. Verify that aws-elasticbeanstalk-service-role has been auto-selected for the service role.
9. Verify that aws-elasticbeanstalk-ec2-role has been auto-selected for the instance profile.
10. Click "Skip to review" button.
11. Click the "Submit" button.

## RDS Database Creation

1. Go to AWS Management Console and use Find Services to search for RDS
2. Click Create database button
3. Select PostgreSQL
4. In Templates, check the Free tier box.
5. Scroll down to Settings.
6. Set DB Instance identifier to multi-docker-postgres
7. Set Master Username to postgres
8. Set Master Password to postgrespassword and confirm.
9. Scroll down to Connectivity. Make sure VPC is set to Default VPC
10. Scroll down to Additional Configuration and click to unhide.
11. Set Initial database name to fibvalues
12. Scroll down and click Create Database button

## ElastiCache Redis Creation

1. Go to AWS Management Console and use Find Services to search for ElastiCache
2. In the sidebar under Resources, click Redis Clusters
3. Click the Create Redis cluster button
4. Make sure Cluster Mode is DISABLED.
5. Scroll down to Cluster info and set Name to multi-docker-redis
6. Scroll down to Cluster settings and change Node type to cache.t2.micro
7. Change Number of Replicas to 0 (Ignore the warning about Multi-AZ)
8. Scroll down to Subnet group. Select Create a new subnet group if not already selected.
9. Enter a name for the Subnet Group such as redis.
10. Scroll down and click the Next button
11. Scroll down and click the Next button again.
12. Scroll down and click the Create button.

## Creating a Custom Security Group

1. Go to AWS Management Console and use Find Services to search for VPC
2. Find the Security section in the left sidebar and click Security Groups
3. Click Create Security Group button
4. Set Security group name to multi-docker
5. Set Description to multi-docker
6. Make sure VPC is set to your default VPC
7. Scroll down and click the Create Security Group button.
8. After the security group has been created, find the Edit inbound rules button.
9. Click Add Rule
10. Set Port Range to 5432-6379
11. Click in the box next to Source and start typing 'sg' into the box. Select the Security Group you just created.
12. Click the Save rules button

## Applying Security Groups to ElastiCache

1. Go to AWS Management Console and use Find Services to search for ElastiCache
2. Under Resources, click Redis clusters in Sidebar
3. Check the box next to your Redis cluster
4. Click Actions and click Modify
5. Scroll down to find Selected security groups and click Manage
6. Tick the box next to the new multi-docker group and click Choose
7. Scroll down and click Preview Changes
8. Click the Modify button.

## Applying Security Groups to RDS

1. Go to AWS Management Console and use Find Services to search for RDS
2. Click Databases in Sidebar and check the box next to your instance
3. Click Modify button
4. Scroll down to Connectivity and add select the new multi-docker security group
5. Scroll down and click the Continue button
6. Click Modify DB instance button

## Applying Security Groups to Elastic Beanstalk

1. Go to AWS Management Console and use Find Services to search for Elastic Beanstalk
2. Click Environments in the left sidebar.
3. Click MultiDocker-env
4. Click Configuration
5. In the Instances row, click the Edit button.
6. Scroll down to EC2 Security Groups and tick the box next to multi-docker
7. Click Apply and Click Confirm

## Setting Environment Variables

1. Go to AWS Management Console and use Find Services to search for Elastic Beanstalk
2. Click Environments in the left sidebar.
3. Click MultiDocker-env
4. Click Configuration
5. In the Software row, click the Edit button
6. Scroll down to Environment properties
7. Set REDIS_HOST key to the primary endpoint listed above, remember to omit :6379
8. Set REDIS_PORT to 6379
9. Set PGUSER to postgres
10. Set PGPASSWORD to postgrespassword
11. Set the PGHOST key to the endpoint value listed above.
12. Set PGDATABASE to fibvalues
13. Set PGPORT to 5432
14. Click Apply button

## IAM Keys for Deployment

1. Search for the "IAM Security, Identity & Compliance Service"
2. Click "Create Individual IAM Users" and click "Manage Users"
3. Click "Add User"
4. Enter any name you’d like in the "User Name" field.
5. Click "Next"
6. Click "Attach Policies Directly"
7. Search for "beanstalk"
8. Tick the box next to "AdministratorAccess-AWSElasticBeanstalk"
9. Click "Next"
10. Click "Create user"
11. Select the IAM user that was just created from the list of users
12. Click "Security Credentials"
13. Scroll down to find "Access Keys"
14. Click "Create access key"
15. Select "Command Line Interface (CLI)"
16. Scroll down and tick the "I understand..." check box and click "Next"

## AWS Keys in Travis

1. Go to your Travis Dashboard and find the project repository for the application we are working on.
2. On the repository page, click "More Options" and then "Settings"
3. Create an AWS_ACCESS_KEY variable and paste your IAM access key
4. Create an AWS_SECRET_KEY variable and paste your IAM secret key

## Deploying App

1. Make a small change to your src/App.js file in the greeting text.
2. In the project root, in your terminal run:

```
git add.
git commit -m “testing deployment"
git push origin main

```

3. Go to your Travis Dashboard and check the status of your build. The status should eventually return with a green checkmark and show "build passing"
4. Go to your AWS Elastic Beanstalk application (It should say "Elastic Beanstalk is updating your environment")

**It should eventually show a green checkmark under "Health". You will now be able to access your application at the external URL provided under the environment name.**
