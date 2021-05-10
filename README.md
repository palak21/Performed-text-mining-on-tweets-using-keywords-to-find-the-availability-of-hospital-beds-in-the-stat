# AWS-Bigdata using Kinesis , Elastic Search and Kibana.

India is reporting more than 250,000 new COVID-19 cases a day in its worst phase of the pandemic. Hospitals are turning away patients and supplies of oxygen and medication are running short.
In response, people are bypassing the conventional lines of communication and turning to Twitter to crowdsource help for oxygen cylinders, hospital beds and other requirements.
People in need and those with information or resources share telephone numbers of volunteers, vendors who have oxygen cylinders or drugs, and details of which medical facility can take patients using hashtags like #COVIDSOS. Twitter is not as widely used in India as Facebook or WhatsApp but it is proving a more valuable tool to get pleas for help out in the coronavirus crisis, largely because of its "re-tweet" function that can quickly amplify a message through users' networks of contacts.

We created an analysis that will filter tweets containing keywords such as [ Urgently, beds required, bed, need, delhi] while also displaying the most up-to-date information on available beds from the https://coronabeds.jantasamvad.org/covid-info.js website.

Architecture Overview:

![image](https://user-images.githubusercontent.com/34096576/117728033-b9325f00-b19d-11eb-9545-98468e4243c0.png)

Prerequisites:
(please read these carefully and do not jump ahead and start setting things up unless specifically called out)

AWS Account:
In order to work on this project you'll need an AWS Account with access to create AWS IAM, S3, Kinesis, Elastic Search and API Kibana. 

Steps:
Step1: Create an AWS Account
Step2: Start an EC2 Instance
The steps outlined in this tutorial assume that you are using an EC2 instance as the
web server and log producer. (For detailed instructions, see Getting started with
Amazon EC2 Linux instances.)
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/
2. From the console dashboard, choose Launch Instance. 
3. On the Choose an Amazon Machine Image (AMI) page, choose Amazon
Linux AMI.
4. On the Choose an Instance Type page, select the t2.micro instance type.
Figure 2: Choose an instance type
5. Choose Next: Configure Instance Details.
6. Choose Create new IAM role. A new tab opens to create the role.
Figure 3: Configure instance details
7.You want to ensure that your EC2 instance has an AWS Identity and Access
Management (IAM) role configured with permission to write to Amazon Kinesis
Data Firehose and Amazon CloudWatch. For more information, see IAM Roles
for Amazon EC2.
a. Choose Create role.
b. For trusted entity, choose AWS service.
c. For the use case, choose EC2.
Figure 4: Create new IAM role
Amazon Web Services Build a Log Analytics Solution on AWS
10
d. Choose Next: Permissions.
e. In the search bar, type KinesisFirehose and select the check box for
AmazonKinesisFirehoseFullAccess.
Figure 5: Add AmazonKinesisFirehoseFullAccess policy
7. Clear the search bar and type CloudWatchFull. Select the check box for
CloudWatchFullAccess.
Figure 6: Add CloudWatchFullAccess policy
f. Choose Next: Tags to add optional tags.
g. Choose Next: Review and for Role name, type web-log-ec2-role.
h. Choose Create role.
8. Return to the EC2 launch wizard tab (Figure 3) and next to the IAM role, click
the refresh icon. Then, select web-log-ec2-role.
Amazon Web Services Build a Log Analytics Solution on AWS
10. Choose Review and Launch
11. Review the details and choose Launch.
12. In the key pair dialog box that appears, choose to create a new key pair or
select an existing key pair that you have access to. If you choose to create a
new key pair, choose Download Key Pair and wait for the file to download.
Amazon Web Services Build a Log Analytics Solution on AWS
12
Figure 8: Create a new key pair
13. Choose Launch Instances.
The EC2 instance launches with the required dependencies already installed on the
machine. The user data script clones Github onto the EC2 instance and the required file
is copied into a new directory named logs in the /tmp folder. Once the EC2 instance is
launched, you need to connect to it via SSH.
