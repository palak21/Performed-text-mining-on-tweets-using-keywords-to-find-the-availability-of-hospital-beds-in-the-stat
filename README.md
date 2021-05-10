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
Step1: **Create an AWS Account**


Step2: **Start an EC2 Instance**
The steps outlined in this tutorial assume that you are using an EC2 instance as the
web server and tweet producer. (For detailed instructions, see Getting started with
Amazon EC2 Linux instances.)
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/

2. From the console dashboard, choose Launch Instance. 

3. On the Choose an Amazon Machine Image (AMI) page, choose Amazon
Linux AMI.

4. On the Choose an Instance Type page, select the t2.micro instance type.

![image](https://user-images.githubusercontent.com/34096576/117732459-c9017180-b1a4-11eb-8b23-f89ef7f6dcdd.png)
                                            Figure 2: Choose an instance type
5. Choose Next: Configure Instance Details.

![image](https://user-images.githubusercontent.com/34096576/117732773-47f6aa00-b1a5-11eb-9c38-776979d07e82.png)
                                           Figure 3: Configure instance details
6. Choose Create new IAM role. A new tab opens to create the role.

7.You want to ensure that your EC2 instance has an AWS Identity and Access
Management (IAM) role configured with permission to write to Amazon Kinesis
Data Firehose and Amazon CloudWatch. For more information, see IAM Roles
for Amazon EC2.
a. Choose Create role.
b. For trusted entity, choose AWS service.
c. For the use case, choose EC2.

![image](https://user-images.githubusercontent.com/34096576/117733206-061a3380-b1a6-11eb-854b-5f08fdea3e39.png)

                              Figure 4: Create new IAM role

                                
d. Choose Next: Permissions.
e. In the search bar, type KinesisFirehose and select the check box for
AmazonKinesisFirehoseFullAccess.
 ![image](https://user-images.githubusercontent.com/34096576/117732989-aae84100-b1a5-11eb-9697-c3dc70c5ca92.png)
 
                       Figure 5: Add AmazonKinesisFirehoseFullAccess policy
f. Clear the search bar and type CloudWatchFull. Select the check box for
CloudWatchFullAccess.
g. Choose Next: Tags to add optional tags.
h. Choose Next: Review and for Role name, type tweet-search-role.
i. Choose Create role.


8. Return to the EC2 launch wizard tab (Figure 3) and next to the IAM role, click
the refresh icon. Then, select web-log-ec2-role.
Amazon Web Services Build a Log Analytics Solution on AWS

9. Choose Review and Launch

10. Review the details and choose Launch.

11. In the key pair dialog box that appears, choose to create a new key pair or
select an existing key pair that you have access to. If you choose to create a
new key pair, choose Download Key Pair and wait for the file to download.
Amazon Web Services Build a Log Analytics Solution on AWS


12. Choose Launch Instances.
The EC2 instance launches with the required dependencies already installed on the
machine. The user data script clones Github onto the EC2 instance and the required file
is copied into a new directory named logs in the /tmp folder. Once the EC2 instance is
launched, you need to connect to it via SSH.


STEP:3 **PUT PYTHON SCRIPT ON EC2 INSTANCE**
Connect to EC2 instance: 
1)ssh -i WebLogsKeypair.pem userName@DNSpublicAddress
13)git clone https://github.com/palak21/AWS-Bigdata-/blob/main/tweeter_final__script.ipynb


STEP:4 **Create an Amazon Kinesis Data Firehose
Delivery Stream**

To create the Amazon Kinesis Data Firehose delivery stream:
14.Open the Amazon Kinesis console at https://console.aws.amazon.com/kinesis.

15. In the Get Started section, choose Kinesis Data Firehose, and then choose
Create Delivery Stream.

16. On the Name and source screen:

a. For Delivery stream name, enter web-log-ingestion-stream.

b. For Choose a source, select Direct PUT or other sources.
c. Choose Next.
17. On the Process records screen, keep the default selections and choose Next.

18. On the Choose a destination screen:
a. For Destination, choose Amazon S3.
Amazon Web Services Build a Log Analytics Solution on AWS

b. For S3 bucket, choose Create new.
c. In the Create S3 bucket window, for S3 bucket name, specify a unique
name. You do not need to use the name elsewhere in this tutorial. However,
Amazon S3 bucket names are required to be globally unique.
d. For Region, choose US East (N. Virginia).
e. Choose Create S3 Bucket.


19. Choose Next.

20. On the Configure settings screen, scroll down to Permissions, and for IAM
role, choose Create or update IAM role.
Figure 9: Permissions – IAM role settings

21. Choose Next.

22. Review the details of the Amazon Kinesis Data Firehose delivery stream and
choose Create Delivery Stream. 
  
  
Step 3: **Install and Configure the Amazon Kinesis
Agent on the EC2 Instance**

Now that you have an Amazon Kinesis Firehose delivery stream ready to ingest your
data, you can configure the EC2 instance to send the data using the Amazon Kinesis
Agent software. 

The agent is a standalone Java software application that offers an easy
way to collect and send data to Kinesis Data Firehose. The agent continuously monitors
a set of files and sends new data to your delivery stream. 

It also emits Amazon CloudWatch metrics to help you better
monitor and troubleshoot the streaming process.

1. To install the agent, copy and paste the following command. For more
information, see Download and Install the Agent.


`sudo yum install –y aws-kinesis-agent`

2. For detailed instructions on how to configure the agent to process and send log
data to your Amazon Kinesis Data Firehose delivery stream, see Configure and
Start the Agent.

To configure the agent for this tutorial, modify the configuration file located at
`/etc/aws-kinesis/agent.json` using the following template.

o Replace filePattern with the full-path-to-log-file that represents the path
to your log files and a wildcard if you have multiple log files with the same
naming convention. For example, it might look similar to:
“/tmp/logs/access_log*”


o Replace name-of-delivery-stream with the name of the Kinesis Data
Firehose delivery stream you created in Step 2.
o The firehose.endpoint is firehose.us-east-1.amazonaws.com
(default).
 `sudo vi /etc/aws-kinesis/agent.json`



3. Start the agent manually by issuing the following command:
`sudo service aws-kinesis-agent start`

![image](https://user-images.githubusercontent.com/34096576/117735543-7e82f380-b1aa-11eb-8e0c-bcd68cc42323.png)


Once started, the agent looks for files in the configured location and send the records to
the Kinesis Data Firehose delivery stream. 
