# Automated-Receipt-Tracker using AWS

  
### Project Overview

  
This project provides an automated solution for processing receipts using Amazon Web Services (AWS). Traditional manual receipt handling is often time-consuming, prone to errors, and does not scale effectively.
  
This system addresses these challenges by automatically extracting structured data from receipts. The data is then stored efficiently in a centralized location, streamlining record-keeping and simplifying auditing processes.
  
Excellent, this clearly outlines the components. Let’s refine this to better explain how the components interact.
  
Here are two improved versions.
  
Architecture
  
The project’s architecture is composed of the following AWS services:
  
Amazon S3 (Storage Layer): Stores all incoming receipt files, including images and PDFs. This acts as the entry point for the processing workflow.
  
AWS Lambda (Compute Layer): Provides the serverless compute logic that automates the workflow. Lambda functions are triggered by new files in S3 and coordinate the other services.
  
Amazon Textract (Processing Layer): Performs AI-powered OCR to automatically extract structured data, such as vendor names, dates, and line items, from the receipts.
  
Amazon DynamoDB (Database Layer): A NoSQL database used to store the extracted receipt data in a structured, queryable format for long-term record-keeping.
  
Amazon SES (Notification System): The Simple Email Service is used to send automated email alerts to users, confirming successful receipt processing and providing key details.
  
  
# 1. Set Up S3 Buckets for Receipt Storage and Archiving
The receipt processing system needs a central, secure location to store both the original receipt files and track their processing status.
  
Amazon S3 (Simple Storage Service) provides the ideal solution with its durability, availability, and security features. This bucket will serve as both the entry point for new receipts and the archive for processed documents.
  
Steps
  
Sign in to the AWS Management Console
  
Navigate to the Amazon S3 service
  
Creating the S3 Bucket
  
Click the Create bucket button to open the bucket creation wizard.
  
Provide a unique and descriptive name for your bucket (e.g., your-project-receipts). This bucket will store all your receipt files.
  
In the Bucket type section, ensure that General purpose is selected.
  

Note: S3 Buckets Must Be Globally Unique
  
Your bucket name must be unique across all existing bucket names in all of AWS. To ensure uniqueness, it’s a best practice to add a personal identifier to the name.
  
Example: automated-receipts-yourusername
  
❗Important: Keep all default settings for the bucket configuration

Finalize Bucket Creation
  
Finally, scroll to the bottom of the page and click the Create bucket button. AWS will validate your settings and create the bucket, after which you will be redirected to the S3 console where your new bucket will appear in the list.
  
Create an “incoming” Folder (Recommended)
  
To keep files organized by their processing status, we recommend creating a dedicated folder for new receipt uploads.

Navigate into your bucket: From the S3 console, click on the name of the bucket you just created.
  
Initiate folder creation: Click the Create folder button.
  
Name the folder: In the Folder name field, enter incoming and then click Create folder at the bottom of the page.


## Role of the S3 Bucket
  
The Amazon S3 bucket is the foundation of the workflow, serving a dual role as both the trigger for processing and the long-term archive for all receipts.
  
The process begins when a user or automated system uploads a receipt image or PDF into the incoming/ folder. This upload event automatically triggers an AWS Lambda function, which initiates the data extraction. 

Throughout this process, the original receipt file is securely preserved in the bucket, creating a permanent and auditable record for compliance and verification purposes.

# 2. Create a DynamoDB Table for Receipt Data
  
Once the system extracts structured data from a receipt, it requires a durable and high-performance database to store that information. For this purpose, we will use Amazon DynamoDB.
  
Why DynamoDB?
  
DynamoDB is an ideal choice for this serverless application for several key reasons:
  
Serverless: It is a fully managed service, which means there is no need to provision or manage servers, patch software, or handle other administrative tasks.
  
Scalability: It scales seamlessly to handle virtually any amount of data or traffic, ensuring the application remains responsive as usage grows.
  
Performance: It delivers consistent, single-digit millisecond latency, which is essential for a responsive system.
  
Steps
  
Navigate to DynamoDB in the AWS Console
  
## Create the Table
  
Click the Create table button to open the creation interface. This table will store all the structured data extracted from your receipts.
  
  
1. Table name
  
Enter Receipts.
  
2. Partition key
  
Enter receipt_id for the key name and select String as the key type.
  
Note: The partition key is a unique identifier for each receipt stored in the table. This ID will be generated automatically by the processing Lambda function.

3. Sort key
  
Enter date for the key name and select String as the key type.
  
Note: Adding a sort key allows for more powerful queries, such as retrieving all receipts within a specific date range. We’ll use a YYYY-MM-DD format to ensure it sorts correctly.
  

Review and Create Table
  
Accept default settings: You can leave all other settings below the sort key with their default values.
  
Create the table: Scroll to the bottom of the page and click the Create table button.
  
AWS will now provision your DynamoDB table, which is typically ready in under a minute. You will be redirected to the tables dashboard, where you’ll see your new Receipts table in the list.



Understanding the Initial Table State
  
After creating the Receipts table, you will see it listed in the DynamoDB console with a status of Active and showing 0 items. This is the correct and expected state.
  
The table will only be populated with data after the complete processing workflow is triggered. An empty table simply confirms that it has been configured correctly and is ready to receive data once the system is running.
  
Data Model for Receipt Information
  
The DynamoDB table will store the following attributes for each receipt:
  

The DynamoDB table serves as the structured data repository for all processed receipts:
  
The Lambda function extracts data from receipts using Amazon Textract
  
The extracted data is formatted according to the data model
  
The Lambda function writes this data to the DynamoDB table
  
Applications and reports can query the table for receipt information
  
The original receipt images remain in S3, linked via the receipt_url attribute
  
# Notification Setup: Configuring Amazon SES
  
Steps to be Performed
  
Setting up SES for Email Notifications.
  
Verifying the Email Address.
  
Configuring SES for Receipt Notifications.
  
## 1. Setting up SES for Email Notifications
  
After receipts are processed, users need to be notified about the extracted information and any actions required.
  
Amazon Simple Email Service (SES) provides a reliable, cost-effective email solution for sending these notifications. Setting up SES ensures that appropriate stakeholders are informed when new receipts are processed.

Navigate to Amazon SES in the AWS Console

## Access Identity Management
  
In the SES console, go to Configuration > Identities
  
This section is where you’ll verify email addresses or domains

Begin Identity Creation
  
Click the Create identity button
  
This starts the process of verifying an email address or domain for sending

Select Identity Type
  
Select Email address option
  
Note: For production systems, you might want to verify an entire domain instead

Configure Sender Email
  
Enter your sender email address (the address you’ll send notifications from)
  
Best practice: Use a business email or a dedicated notification address
  
Example: receipts-noreply@yourcompany.com
  
Complete Initial Setup
  
Click Create identity
  
AWS will process your request and prepare verification


## 2. Verifying the Email Address
  
After creating the identity, navigate to your Gmail inbox and perform the verification steps as provided below:

Verify Sender Email
  
AWS will send a verification email to the address you provided
  
This verification ensures you own the email address
  
Check both inbox and spam/junk folders for this email


Complete Verification
  
Go to the inbox of that email address and find the AWS verification email
  
Click the verification link in that email


You’ll be redirected to a confirmation page in AWS

## 3. Configuring SES for Receipt Notifications
  
❗Important: Email Verification Options

Option 1: Use the same email for sender and recipient (simplest approach)
  
Verify only one email address to use for both sending and receiving notifications
  
This is recommended for tutorial and testing purposes
  
Option 2: Use different sender and recipient emails
  
Repeat steps 1 and 2 for each recipient email address
  
Remember: While in SES sandbox mode, both sender and recipient emails must be verified


The SES configuration enables the notification component of the receipt processing workflow:

After the Lambda function processes a receipt and stores data in DynamoDB
  
It uses SES to send an email notification with receipt details
  
The notification includes key information like vendor, date, amount, and a link to the original receipt
  
Users receive timely updates without needing to check the system manually
  
# Processing Setup: Creating a Lambda function
  
Steps to be Performed
  
Create an IAM role for AWS Lambda.
  
Create a Lambda Function.
  
Add the Lambda function Code.
  
Understand the Lambda function Code.
  
## 1. Create an IAM role
  
Lambda functions need permission to access other AWS services.

Creating a dedicated IAM role follows security best practices by explicitly defining what actions your Lambda function can perform. To do this —

Navigate to IAM in the AWS Console

Initiate Role Creation
  
Click “Roles” in the left navigation pane
  
Click “Create role” button to start the process

Select Trusted Entity
  
Select “AWS service” as the trusted entity type
  
This specifies which AWS service can assume this role
  


Specify Service Use Case
  
Choose “Lambda” as the service that will use this role
  
This allows Lambda functions to assume this role at runtime


Configure Permissions
Click “Next: Permissions” to continue
On the Permissions policies page, you’ll attach policies that define what the role can do


Attach Required Policies
  
In the search box, search for and select these policies:
  
AmazonS3ReadOnlyAccess: Allows reading from S3 buckets
  
AmazonTextractFullAccess: Enables document analysis with Textract
  
AmazonDynamoDBFullAccess: Permits writing extracted data to DynamoDB
  
AmazonSESFullAccess: Enables sending email notifications
  
AWSLambdaBasicExecutionRole: Allows Lambda to write logs to CloudWatch


Review and Name Role
  
Click “Next”
  
Role name: “ReceiptProcessingLambdaRole”
  
Description (optional): “Allows Lambda functions to process receipts using S3, Textract, DynamoDB, and SES”


Create the Role
  
Review the role configuration
  
Click “Create role” to finalize


Validate Role Creation
  
Verify the role appears in the IAM roles list
  
Click on the role name to view its details and ensure all policies are attached correctly


The IAM role connects all components of the receipt processing system:

When the Lambda function executes, it assumes this role
  
The role’s permissions allow the function to:

Read receipt files from the S3 bucket
  
Send those files to Textract for analysis
  
Write extracted data to DynamoDB
  
Send notification emails via SES
  
Write logs to CloudWatch for troubleshooting
  
## 2. Create a Lambda Function
  
The Lambda function serves as the central processing engine for our receipt processing system.

This serverless function is triggered automatically when a new receipt is uploaded to S3, coordinates the extraction of data using Textract, stores the structured information in DynamoDB, and sends notifications via SES.

Navigate to Lambda in the AWS Console


Begin Function Creation
  
Click “Create function” to start the process
  
This opens the function creation wizard


Select Creation Method
  
Select “Author from scratch”
  
This allows you to create a custom function from the beginning
  
Configure Basic Settings

Function name: Enter “ReceiptProcessor”
  
Runtime: Select “Python 3.10” from the dropdown
  
Architecture: Leave as default (x86_64)


Set Permissions
  
Under “Permissions” expand “Change default execution role”
  
Select “Use an existing role”
  
From the dropdown, choose “ReceiptProcessingLambdaRole”


Create the Function
  
Review your settings
  
Click “Create function”
  
AWS will provision your Lambda function (this takes just a few seconds)


Adjust Function Timeout
  
Navigate to the “Configuration” tab

Select “General configuration”
  
Click “Edit”


Change the Timeout from the default 3 seconds to 3 minutes (180 seconds)
  
This extended timeout is necessary because Textract processing can take time for complex receipts
  
Click “Save”


Configure Environment Variables
  
Still in the “Configuration” tab, select “Environment variables”
  
Click “Edit”


Add the following key-value pairs:
  
Key: DYNAMODB_TABLE, Value: Receipts
  
Key: SES_SENDER_EMAIL, Value: your-verified-email@example.com (use the email you verified in SES)
  
Key: SES_RECIPIENT_EMAIL, Value: recipient-email@example.com (use the recipient email you verified)
  
Note: For detailed information about these environment variables and their usage in the Lambda function, please refer to the Lambda Environment Variables table given below.
Click Save.

Lambda Environment Variables:

Environment variables provide configuration without code changes:

The Lambda function forms the core of the receipt processing workflow:

When a new receipt is uploaded to S3, it triggers the Lambda function
  
Lambda retrieves the receipt file from S3
  
Lambda calls Amazon Textract to extract data from the receipt
  
Lambda processes and structures the extracted data
  
Lambda writes the structured data to DynamoDB
  
Lambda sends a notification email via SES
  
Lambda moves the original receipt to the “processed” folder

## 3. Add the Lambda function Code
  
Access the Code Editor
  
In the Lambda console, navigate to your “ReceiptProcessor” function
  
Scroll down to the code source section where you’ll see the code editor
  
The default Lambda function contains a simple “Hello World” example


Replace the default code with the provided Python code
  
Delete the existing code in the editor
  
Copy and paste the Python code provided in this repository

# Integration and Testing
  
Steps to be Performed
  
Setup S3 Event Notification Trigger.
  
Test the Project Execution.
  
Project Improvement Ideas.
  
## 1. Setup S3 Event Notification Trigger
  
For our receipt processing system to operate automatically, we need to establish a trigger that will invoke our Lambda function whenever a new receipt is uploaded.

Amazon S3 offers event notifications that can detect new file uploads and trigger specific actions. This integration creates the vital connection between file upload (input) and processing (execution).

Navigate to S3 in the AWS Console

Access Your Bucket
  
From the list of buckets, select the receipt storage bucket you created earlier
  
This opens the bucket management interface


Access Properties Settings
  
Navigate to the “Properties” tab at the top of the bucket management interface
  
This tab contains various bucket configurations


Create Event Notification
  
Scroll down to the “Event Notifications” section
  
Click “Create event notification” to begin configuration
  
This opens the event notification creation form


Configure Event Details
  
Name: “ReceiptUploadEvent”
  
Prefix (optional): “incoming/” (if using folders)
  
Suffix (optional): Leave blank or add “.pdf,.jpg,.jpeg,.png”


Event types: Check “All object create events”
  
This includes put, post, copy, and multipart upload completions
  
Keep the rest of the boxes unchecked
  
Ensures any method of adding files triggers the process


Destination: Select “Lambda Function”
  
Lambda function: Select “ReceiptProcessor”
  
This connects the S3 event to your processing function
  
Save the Configuration
  
Review all settings to ensure they match your requirements
  
Click “Save” to activate the event notification
  
AWS will validate and create the notification configuration


The S3 event notification creates the automated workflow for receipt processing:

When a receipt image is uploaded to S3, it generates an event
  
The event notification system detects this upload event
  
If the file matches the configured prefix and suffix filters, S3 invokes the Lambda function
  
The Lambda function receives event data including the bucket name and file key
  
Processing begins automatically without any manual intervention
  
## 2. Test the Project Execution
  
This point explains how to verify your receipt processing system works correctly end-to-end.

## Step 1: Upload a Test Receipt

Navigate to your S3 bucket in the AWS Console


Upload a receipt from this folder: Receipts (to the “incoming” folder if you created one)

Wait 10–15 seconds for processing to complete
  
## Step 2: Monitor the Lambda Execution

Go to Lambda > Functions > ReceiptProcessor
Select the “Monitor” tab


Check that your function was recently invoked
  
📝Note: It may take a few minutes for the Lambda execution to appear in the monitoring dashboard. If you don’t see your execution immediately, wait 2–3 minutes and refresh the page.

❗Important: Uploading a receipt in Step 1 is mandatory to trigger the Lambda function execution.

Click “View logs in CloudWatch” for detailed execution logs

Log entries showing processing steps

## Step 3: Verify Data in DynamoDB

Go to DynamoDB > Tables > Receipts
Select the “Items” tab

Look for your recently processed receipt
Check:

Receipt data is stored correctly
Key fields (vendor, date, total amount) are accurate

## Step 4: Check Email Notifications

Open your configured recipient email account

Of course. Here is a more structured and descriptive version of your project enhancement list.
Project Enhancements
Here are several ways to expand upon the core functionality of your receipt processing system, grouped by theme.

User Interface & Experience

  
Implement a Web Upload Form Host a static HTML page on Amazon S3 with a simple form, allowing users to upload receipts directly through a browser instead of just using the S3 console.

Track Processing Status Add a status attribute to your DynamoDB items. Update it throughout the workflow with values like Pending, Processing, Success, or Failed to provide clear feedback on a receipt's state.
  
Core Features & Data
Add Expense Categorization Modify the DynamoDB table and Lambda function to include an expense category field. This allows for better organization of spending into buckets like "Food," "Travel," or "Office Supplies."
  
Generate Monthly Reports Create a new Lambda function that runs on a schedule (e.g., using Amazon EventBridge). This function can query DynamoDB for the month’s expenses, generate a summary, and email it to a user via SES.
  
Operations & Efficiency
Set Up Error Notifications Use Amazon SNS (Simple Notification Service) to send an immediate alert to an administrator if the processing Lambda function encounters an error. This enables faster troubleshooting.
Optimize Image Storage Add a step to your Lambda function to compress or resize receipt images before they are archived. This can significantly reduce long-term S3 storage costs.
Security
  
Introduce User Authentication Integrate Amazon Cognito to manage user accounts. This adds a secure sign-up and sign-in layer to your web interface, ensuring that only authorized users can upload and view receipts.
