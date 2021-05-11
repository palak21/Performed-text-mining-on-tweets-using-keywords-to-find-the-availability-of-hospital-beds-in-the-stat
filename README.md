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
 -Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/

![image](https://user-images.githubusercontent.com/34096576/117732459-c9017180-b1a4-11eb-8b23-f89ef7f6dcdd.png)
                                            Figure 2: Choose an instance type


![image](https://user-images.githubusercontent.com/34096576/117732773-47f6aa00-b1a5-11eb-9c38-776979d07e82.png)
                                           Figure 3: Configure instance 

You want to ensure that your EC2 instance has an AWS Identity and Access
Management (IAM) role configured with permission to write to Amazon Kinesis
Data Firehose and Amazon CloudWatch. For more information, see IAM Roles
for Amazon EC2.

![image](https://user-images.githubusercontent.com/34096576/117733206-061a3380-b1a6-11eb-854b-5f08fdea3e39.png)

                              Figure 4: Create new IAM role


 ![image](https://user-images.githubusercontent.com/34096576/117732989-aae84100-b1a5-11eb-9697-c3dc70c5ca92.png)
 
                       Figure 5: Add AmazonKinesisFirehoseFullAccess policy


In the key pair dialog box that appears, choose to create a new key pair or
select an existing key pair that you have access to. If you choose to create a
new key pair, choose Download Key Pair and wait for the file to download.
Amazon Web Services Build a Log Analytics Solution on AWS


Choose Launch Instances.
The EC2 instance launches with the required dependencies already installed on the
machine. The user data script clones Github onto the EC2 instance and the required file
is copied into a new directory named logs in the /tmp folder. Once the EC2 instance is
launched, you need to connect to it via SSH.


STEP:3 **PUT PYTHON SCRIPT ON EC2 INSTANCE**
Connect to EC2 instance: 

1)ssh -i WebLogsKeypair.pem userName@DNSpublicAddress
2)git clone https://github.com/palak21/AWS-Bigdata-/blob/main/tweeter_final__script.ipynb
3)git clone https://github.com/palak21/AWS-Bigdata-/blob/main/Available_Beds.py


STEP:4 **Create an Amazon Kinesis Data Firehose
Delivery Stream**

To create the Amazon Kinesis Data Firehose delivery stream:
Open the Amazon Kinesis console at https://console.aws.amazon.com/kinesis.

For Destination, choose Amazon S3.

Review the details of the Amazon Kinesis Data Firehose delivery stream and
choose Create Delivery Stream. 
  
  
Step 3: **Install and Configure the Amazon Kinesis
Agent on the EC2 Instance**

Now that you have an Amazon Kinesis Firehose delivery stream ready to ingest your
data, you can configure the EC2 instance to send the data using the Amazon Kinesis
Agent software. 

The agent is a standalone Java software application that offers an easy
way to collect and send data to Kinesis Data Firehose. The agent continuously monitors
a set of files and sends new data to your delivery stream. 


To install the agent, copy and paste the following command. For more
information, see Download and Install the Agent.


`sudo yum install –y aws-kinesis-agent`


To configure the agent for this tutorial, modify the configuration file located at
`/etc/aws-kinesis/agent.json` using the following template.


o Replace name-of-delivery-stream with the name of the Kinesis Data
Firehose delivery stream you created in Step 2.
o The firehose.endpoint is firehose.us-east-1.amazonaws.com
(default).
 `sudo vi /etc/aws-kinesis/agent.json`


Start the agent manually by issuing the following command:
`sudo service aws-kinesis-agent start`

![image](https://user-images.githubusercontent.com/34096576/117735543-7e82f380-b1aa-11eb-8e0c-bcd68cc42323.png)


Once started, the agent looks for files in the configured location and send the records to
the Kinesis Data Firehose delivery stream. 

Step 4: Create an Amazon Elasticsearch Service
Domain
The data produced by this project is stored in Amazon Elasticsearch Service for later
visualization and analysis. To create the Amazon Elasticsearch Service domain:
Open the Amazon Elasticsearch Service console at
https://console.aws.amazon.com/es.

Create a new domain.

 On the Choose deployment type page, for Deployment type, choose a
Development and testing. For Elasticsearch version, leave it set to the
default value.
 ![image](https://user-images.githubusercontent.com/34096576/117736499-9491b380-b1ac-11eb-9d1b-8bb3023a2834.png)


On the Configure domain page, for Elasticsearch domain name, type available-beds-summary.

![image](https://user-images.githubusercontent.com/34096576/117736627-eb978880-b1ac-11eb-8a23-ec3ee9698696.png)


Step 5: **Create a Second Amazon Kinesis Data
Firehose Delivery Stream**

Now that you have somewhere to persist the output of your Amazon Kinesis Data
Analytics application, you need a simple way to get your data into your Amazon ES
domain. Amazon Kinesis Data Firehose supports Amazon ES as a destination, so
create a second Firehose delivery stream:

-Open the Amazon Kinesis console at https://console.aws.amazon.com/kinesis.
 Create Delivery Stream.
 
 For Destination, choose Amazon Elasticsearch Service.

-For Elasticsearch domain, choose the domain you created in Step 4.

-For Index, type name.
-In the S3 backup section, for Backup mode, choose Failed Records Only.
- For S3 bucket, choose Create new.
-In the Create S3 bucket window, for S3 bucket name, specify a unique
name. You do not need to use the name elsewhere in this tutorial. However,
Amazon S3 bucket names are required to be globally unique.


Add permissions for your Kinesis Data Firehose delivery stream to access your
Elasticsearch Service cluster:
a. Select the newly created stream and choose the
IAM role.
b. In the IAM window that opens, choose Add inline policy.
c. On the Create policy page, choose Choose a service and search for
Elasticsearch in the search box.
Figure 16: Create policy – choose a service search box
d. Select the check box for All Elasticsearch Service actions.

Step 6: **Create an Amazon Kinesis Data Analytics
Application**
You are now ready to create the Amazon Kinesis Data Analytics application to
aggregate data from your streaming web log data and store it in your Amazon ES
domain. To create the Amazon Kinesis Data Analytics application:
1. Open the Amazon Kinesis Analytics console at
https://console.aws.amazon.com/kinesisanalytics.
 Create new application.
![image](https://user-images.githubusercontent.com/34096576/117738332-a7a68280-b1b0-11eb-80f4-f98b608879a0.png)

CREATE OR REPLACE STREAM "DESTINATION_STREAM_NEW_001" (
                         Hospital_Name VARCHAR(255), 
                         Avg_Beds_Vacant   REAL,
                         Avg_Beds_Occupied   REAL,
                         Hospital_Type VARCHAR(32),
                         Last_Updated  VARCHAR(32),
                         Total_Beds    VARCHAR(8),
                         Occupied      Integer);
CREATE OR REPLACE PUMP "STREAM_PUMP" AS 
   INSERT INTO "DESTINATION_STREAM_NEW_001"
     SELECT STREAM "name",
                   avg("vacant") OVER W1 AS Avg_Beds_Vacant,
                   avg("occupied") OVER W1 AS Avg_Beds_Occupied,
                   "type",
                   "last_updated_at",
                   "total",
                   "occupied"
     FROM   "SOURCE_SQL_STREAM_001"
     WINDOW W1 AS (
        PARTITION BY "name" 
        RANGE INTERVAL '24' HOUR PRECEDING);

The code creates a STREAM and a PUMP:
o A stream (in-application) is a continuously updated entity that you can
SELECT from and INSERT into (like a TABLE).
o A pump is an entity used to continuously ‘SELECT...FROM’ a source
STREAM and INSERT SQL results into an output STREAM.
Finally, an output stream can be used to send results into a destination.

Under Destination, choose Kinesis Firehose delivery stream and select the
twitter-aggregated-data stream that you created in Step 5.

After approximately 5 minutes, the output of the SQL statement in your Amazon Kinesis
Data Analytics application will be written to your Amazon ES domain. Amazon ES has
built-in support for Kibana, a tool that allows users to explore and visualize the data
stored in an Elasticsearch cluster. To view the output of your Amazon Kinesis Analytics
application in Kibana: 

1. Open the Amazon ES console at https://console.aws.amazon.com/es.
2. In the Domain column, choose the Amazon ES domain called web-logsummary that you created in Step 4.
3. On the Overview tab, click the link next to Kibana.
4. Enter the username and password you created in Step 4.
![image](https://user-images.githubusercontent.com/34096576/117738570-33201380-b1b1-11eb-8a81-20429f77061d.png)


