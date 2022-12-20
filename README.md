Voice Announcement
Allow a Salesforce user to create ad hoc announcements and closures within their Service Cloud Voice contact center.

Demonstrates the Salesforce/Amazon Connector pattern. There are several parts to this pattern.

Salesforce user creates a record of Voice_Announcement__c
An object triggers calls the VoiceAnnouncement.insertAnnouncement
That calls the DynamoDBAPI to create a record in DynamoDB
When a call comes in, a Lambda checks for records within the DynamoDB table and returns that to the contact flow
The contact flow deals with the response (plays the announcement prompt and branches based on whether the call center is closed)
Installation
These steps must be done after Service Cloud Voice has been enabled and the Contact Center has been provisioned

Install the components/package to the Salesforce org
Remote Site Setting
In AWS find the region of your Amazon Connect instance (e.g., us-west-2)
In Salesforce Setup find Remote Site Settings
Click "New Remote Site"
Remote Site Name = "DynamoDB_[REGION]"
Remote Site URL = "https://dynamodb.[REGION].amazonaws.com"
Disable Protocol Security = unchecked
Active = checked
Click "Save"
Setup an IAM User
Go to IAM in AWS
Choose User in the left navigation
Click "Add Users"
User name = "AWSSCV_Utility"
Select AWS credential type = Access Key - Programmatic access
Click "Next"
Choose "Attach existing policies directly"
Choose AmazonDynamoDBFullAccess
Click "Next"
Click "Next: Review"
Click "Create user"
Copy the Access key ID and Secret access key from this page into your a new AWS Connection Metadata record on your Salesforce org
Setup AWS Connection Metadata Record
In Salesforce Setup find Custom Metadata Types
Click "Manage Records" next to the AWS Connections
Click "New"
Create a new record that includes the correct region, access key, secret key and Amazon Connect Instance ID, check Default
DynamoDB Table and Index
Go to DynamoDB in AWS
Choose Tables from the left navigation
Click "Create Table"
Table name = "AWSSCV_Announcement"
Partition key = "Name" (String)
Click "Create table"
Wait a minute or two for the table to be created
Click the table name
Choose Indexes tab
Click "Create index"
Partition key = "Active" (Number)
Index name = "Active-index"
Click "Create index"
You can test the connection between Salesforce and Amazon now. Give yourself the "Voice Announcement Manager" permission set, create a new Voice Announcement record and then check to see whether a corresponding record has been created in DynamoDB.
Lambda Install
Go to Lambda in AWS
Choose Functions from the left navigation
Click "Create function"
Choose "Author from scratch"
Function name = "AWSSCV_GetAnnouncements"
Runtime = "Node.js 16.x"
Architecture = "x86_64"
Click "Create function"
Copy the contents of /lamba/AWSSCV_GetAnnouncements/index.js into the index.js within Code source panel
Set Lambda Environment Variables
Under the Configuration tab, choose Environment variables. Add the following Environment variables:
IndexName = "Active-index"
TableName = "AWSSCV_Announcement"
Timezone = "America/Chicago" (or change to another from this list https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
Click "Deploy"
Set Lambda Permissions
Under the Permissions tab, Execution role, Click the Role name
Under Permissions policies click "Add permission" > "Attach policy"
Check "AmazonDynamoDBFullAccess"
Click "Attach policies"
Now would be a good time to test the Lamdba. Your test event can be any valid JSON even "{}".
Provide Lambda access in Amazon Connect
Go to Amazon Connect in AWS
Click the instance alias of your instance
Choose "Contact flows" from the left navigation
Under AWS Lambda, choose AWSSCV_GetAnnouncements
Click "Add Lamdba Function"
Make Lamdba Call in Contact Flow
Open the contact flow
Add an Invoke AWS Lambda function block
Choose AWSSCV_GetAnnouncements
Click "Add parameter"
Choose "Use attribute"
Destination key = "DialedNumber"
Type = "System"
Attribute = "Dialed Number"
Timeout = 5
Click "Save"
Add a Play prompt block
Choose "Text-to-speech or chat text"
Choose "Use attribute"
Type = "External"
Attribute = "prompt"
Interpret as "SSML"
Click "Save"
Connect the Invoke AWS Lambda block success to Play Prompt
Add a Check contact attributes block
Type = "External"
Attribute = "closed"
Click "Add condition" - Equals = "true"
Click "Save"
Connect the Play prompt Success to the Check contact attributes block
Add a Disconnect block
Connect the Check contact attributes = true to the Disconnect block
Connect the Check contact attributes No Match to the rest of the flow
