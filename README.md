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

Step 4: Create an Amazon Elasticsearch Service
Domain
The data produced by this project is stored in Amazon Elasticsearch Service for later
visualization and analysis. To create the Amazon Elasticsearch Service domain:
1. Open the Amazon Elasticsearch Service console at
https://console.aws.amazon.com/es.


2. Choose Create a new domain.


4. On the Choose deployment type page, for Deployment type, choose a
Development and testing. For Elasticsearch version, leave it set to the
default value.
 ![image](https://user-images.githubusercontent.com/34096576/117736499-9491b380-b1ac-11eb-9d1b-8bb3023a2834.png)



5. Choose Next.


6. On the Configure domain page, for Elasticsearch domain name, type available-beds-summary.

![image](https://user-images.githubusercontent.com/34096576/117736627-eb978880-b1ac-11eb-8a23-ec3ee9698696.png)

7. Leave all settings as their default values and choose Next



Step 5: Create a Second Amazon Kinesis Data
Firehose Delivery Stream
Now that you have somewhere to persist the output of your Amazon Kinesis Data
Analytics application, you need a simple way to get your data into your Amazon ES
domain. Amazon Kinesis Data Firehose supports Amazon ES as a destination, so
create a second Firehose delivery stream:
1. Open the Amazon Kinesis console at https://console.aws.amazon.com/kinesis.
2. Choose Create Delivery Stream.
3. On the Name and source page:
a. For Delivery stream name, enter web-log-aggregated-data.
b. For Choose a source, select Direct PUT or other sources and choose
Next.
4. On the Process records screen, leave the default values and choose Next.
5. On the Choose a destination page:
a. For Destination, choose Amazon Elasticsearch Service.
b. For Elasticsearch domain, choose the domain you created in Step 4. (You
may need to wait until the Amazon ES domain has finished processing. Click
the refresh button periodically to refresh the list of domains).
Amazon Web Services Build a Log Analytics Solution on AWS
20
c. For Index, type request_data.
d. For Index rotation, choose No rotation (default).
e. For Retry duration, leave the default value of 300 seconds.
Figure 13: Amazon Elasticsearch Service destination settings
f. In the S3 backup section, for Backup mode, choose Failed Records Only.
g. For S3 bucket, choose Create new.
h. In the Create S3 bucket window, for S3 bucket name, specify a unique
name. You do not need to use the name elsewhere in this tutorial. However,
Amazon S3 bucket names are required to be globally unique.
i. For Region, choose US East (N. Virginia).
j. Choose Create S3 Bucket.
6. Choose Next.
Amazon Web Services Build a Log Analytics Solution on AWS
21
Figure 14: S3 backup settings
7. On the Configure settings screen, you can leave all fields set to their default
values. However, you will need to choose an IAM role so that Amazon Kinesis
Firehose can write to your Amazon ES domain on your behalf. For IAM role,
choose Create new or choose.
8. Choose Next.
9. Review the details for your Amazon Kinesis Data Firehose delivery stream and
choose Create Delivery Stream.
10. Add permissions for your Kinesis Data Firehose delivery stream to access your
Elasticsearch Service cluster:
a. Select the newly created web-log-aggregated-data stream and choose the
IAM role.
Amazon Web Services Build a Log Analytics Solution on AWS
22
Figure 15: Web-log-aggregated-data stream IAM role
b. In the IAM window that opens, choose Add inline policy.
c. On the Create policy page, choose Choose a service and search for
Elasticsearch in the search box.
Figure 16: Create policy – choose a service search box
d. Select the check box for All Elasticsearch Service actions.
Amazon Web Services Build a Log Analytics Solution on AWS
23
Figure 17: Create policy – select option for All Elasticsearch service actions
e. Under Resources, select the Any check box and choose Review policy,
Figure 18: Create policy – Any resources check box
f. On the Review policy page, name the policy ElasticSearchAccess and
choose Create policy.
Step 6: Create an Amazon Kinesis Data Analytics
Application
You are now ready to create the Amazon Kinesis Data Analytics application to
aggregate data from your streaming web log data and store it in your Amazon ES
domain. To create the Amazon Kinesis Data Analytics application:
1. Open the Amazon Kinesis Analytics console at
https://console.aws.amazon.com/kinesisanalytics.
2. Choose Create new application.
3. For Application name, type web-log-aggregation-tutorial.
4. Leave the Runtime value as the default and choose Create application.
5. To configure the source data for the Amazon Kinesis Data Analytics application,
choose Connect streaming data. 
Amazon Web Services Build a Log Analytics Solution on AWS
24
6. Under Source, choose Kinesis Firehose delivery stream and select web-logingestion-stream that you created in Step 2.
Figure 19: Specify the Kinesis Firehose delivery stream
7. Scroll down to the Schema section and choose Discover schema., Amazon
Kinesis Analytics analyzes the source data in your Kinesis Data Firehose
delivery stream and creates a formatted sample of the input data for your
review: 
Amazon Web Services Build a Log Analytics Solution on AWS
25
Figure 20: Schema discovery
8. Leave all values set to their defaults, and choose Save and continue. You are
taken back to the hub screen for your Amazon Kinesis Data Analytics
application.
9. To create the SQL that analyzes the streaming data, choose Go to SQL editor.
10. When prompted, choose Yes, start application.
After approximately 60 to 90 seconds, the Source data section presents you
with a sample of source data that is flowing into your source delivery stream.
11. In the SQL editor, enter the following SQL code: 
Amazon Web Services Build a Log Analytics Solution on AWS
26
CREATE OR REPLACE STREAM "DESTINATION_SQL_STREAM"
 (datetime TIMESTAMP, status INTEGER, statusCount INTEGER);
-- Create pump to insert into output
CREATE OR REPLACE PUMP "STREAM_PUMP" AS
 INSERT INTO "DESTINATION_SQL_STREAM"
-- Select all columns from source stream
SELECT
 STREAM ROWTIME as datetime,
 "response" as status,
 COUNT(*) AS statusCount
 FROM "SOURCE_SQL_STREAM_001"
 GROUP BY
 "response",
 FLOOR(("SOURCE_SQL_STREAM_001".ROWTIME - TIMESTAMP
'1970-0101
 00:00:00') minute / 1 TO MINUTE);
The code creates a STREAM and a PUMP:
o A stream (in-application) is a continuously updated entity that you can
SELECT from and INSERT into (like a TABLE).
o A pump is an entity used to continuously ‘SELECT...FROM’ a source
STREAM and INSERT SQL results into an output STREAM.
Finally, an output stream can be used to send results into a destination.
12. Choose Save and run SQL. After about 1 minute, Amazon Kinesis Data
Analytics displays the output of the query.
13. To save the running output of the query, choose the Destination tab and
choose Connect to a destination.
Amazon Web Services Build a Log Analytics Solution on AWS
27
Figure 21: Results of query and Destination tab
14. Under Destination, choose Kinesis Firehose delivery stream and select the
web-log-aggregated-data stream that you created in Step 5.
15. For In-application stream, choose DESTINATION_SQL_STREAM.
16. Leave all other options set to their default values and choose Save and
continue.
Step 7: View the Aggregated Streaming Data
After approximately 5 minutes, the output of the SQL statement in your Amazon Kinesis
Data Analytics application will be written to your Amazon ES domain. Amazon ES has
built-in support for Kibana, a tool that allows users to explore and visualize the data
stored in an Elasticsearch cluster. To view the output of your Amazon Kinesis Analytics
application in Kibana: 
Amazon Web Services Build a Log Analytics Solution on AWS
28
1. Open the Amazon ES console at https://console.aws.amazon.com/es.
2. In the Domain column, choose the Amazon ES domain called web-logsummary that you created in Step 4.
3. On the Overview tab, click the link next to Kibana.
4. Enter the username and password you created in Step 4.
Figure 22: Kibana login screen
Because this is the first time you are opening the Kibana application in your
Amazon ES domain, you need to configure it. This configuration includes giving
your user permissions to access the index in Kibana and giving Kinesis Data
Firehose permission to write to the index.
5. In the left toolbar, choose Security and then choose Role Mappings.
Amazon Web Services Build a Log Analytics Solution on AWS
29
Figure 23: Security options in Kibana
6. Verify an Open Distro Security Role named all_access role is listed. If it is not
listed, choose Add (+ sign) and choose the Role all_access. See Set up MultiTenant Kibana Access in Open Distro for Elasticsearch on the AWS Open
Source Blog for more information.
You need to modify the Users associated with this role.
7. Choose Edit.
Figure 24: Role Mappings – Add button (in purple box), Edit button (in red box)
a. Choose Add User and type admin. This is the user you used to log into
Kibana. 
Amazon Web Services Build a Log Analytics Solution on AWS
30
b. Choose Add User and enter the ARN for your AWS account. The ARN can
be found in the IAM console page, under Users. Click into your user
account and copy the User ARN.
c. Repeat this step for the firehose delivery role you created earlier, found
under Roles in the IAM Console.
You should now have three entries listed for the all_access Kibana Role.
Figure 25: Entries for all_access role
d. Choose Submit.
8. Choose the Kibana icon on the top left to return to the Kibana dashboard.
9. Choose explore on my own. Then, choose Connect to your Elasticsearch
index.
Amazon Web Services Build a Log Analytics Solution on AWS
31
Figure 26: Kibana dashboard – connect to Elasticsearch index
10. the Index pattern field, type request_data*. This entry uses Elasticsearch index
name that you created in Step 5.
11. Choose Next step.
Figure 27: Create index pattern
Kibana automatically identifies the DATETIME field in your input data, which
contains time data.
12. Choose Create index pattern.
To visualize the data in your Elasticsearch index, you will create and configure
a line chart that shows how many of each HTTP response type were included in
the source web log data per minute. 
Amazon Web Services Build a Log Analytics Solution on AWS
32
To create the line chart:
a. In the toolbar, choose Visualize, and then choose Create new visualization.
Figure 28: Create new visualization
b. Choose Line chart.
c. For Choose a search, select request_data*.
To configure your chart, you first need to tell Kibana what data to use for the
y-axis:
d. In the metrics section, choose the arrow next to Y-Axis.
e. Under Aggregation, choose Sum.
f. Under Field, choose STATUSCOUNT.
Now you need to configure the x-axis:
g. In the buckets section, select X-axis with the addition button.
h. Under Aggregation, choose Terms.
i. Under Field, choose STATUS.
j. To run the query and view the line chart, choose on the blue “play” button in
the top right-hand corner. 
Amazon Web Services Build a Log Analytics Solution on AWS
33
Figure 29: Configure X-axis values
Figure 30: Query results in Kibana
Amazon Web Services Build a Log Analytics Solution on AWS
34
Step 8: Clean Up
After completing this tutorial, be sure to delete the AWS resources that you created so
that you no longer accrue charges.
Terminate the EC2 Instance
If you created a new EC2 instance to generate a continuous stream of Apache access
log data, you will need to stop or terminate that instance to avoid further charges.
1. Navigate to the EC2 console at https://console.aws.amazon.com/ec2.
2. Select Running Instances, and find the instance you used to generate your
Apache access logs.
3. On the Actions menu, choose the instance, and then choose Instance State,
then Terminate.
4. Read the warning regarding instanc
