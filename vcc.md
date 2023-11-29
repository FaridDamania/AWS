Q1. Build a serverless architecture using AWS Lambda and API Gateway to allow users to upload images, process them (e.g., resize or apply filters), and then retrieve the processed images through the API Gateway. Provide step-by-step instructions for implementing this system. 

Step 1: Create an S3 Bucket.
Go to S3 in the AWS Management Console.
    Create bucket
    Bucket name: vcc.bucket
    AWS Region: Asia Pacific(Mumbai) ap-south-1 (select the AWS Region you want to create your bucket in)
    and keep all other settings as default for now.

Step 2: Create an IAM Role for Lambda
Go to IAM in the AWS Management Console.
AWS Lambda will require permissions to access S3 and other AWS services.
Navigate on "Roles" from the left sidebar, and then click on "Create role".
    Trusted entity type: AWS service
    Use case: Lambda
    Next -> Add permissions
    Permissions policies
    Permissions policies: AmazonS3FullAccess and AWSLambdaExecute
    Next -> Name, review, and create
    Role details
    Role name: image_upr
    Step 1 and Step 2 as it is
    Step 3: Add tags (add tags if needed)
    Create role

Step 3: Create a Lamda Function.
Go to the Lambda in the AWS Management Console.
To process the image this function will be triggered
Create a Lambda function
    Create function
    Author from scratch
	Function name: image_function
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Use an existing role
	Advance settings -> Enable function URL
    Create function

Step 4: Add code to lambda function.
To process the image add the below code in the code editor which is provided on lambda function page.

import boto3
from PIL import Image
import io

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    size = (128, 128)

    Get the object from the event
    response = s3_client.get_object(Bucket=bucket_name, Key=key)
    image = Image.open(response['Body'])

    Resize the image
    image.thumbnail(size)

    Save the image to a buffer
    buffer = io.BytesIO()
    image.save(buffer, 'JPEG')
    buffer.seek(0)

    Put the object back on S3
    new_key = f"resized-{key}"
    s3_client.put_object(Bucket=bucket_name, Key=new_key, Body=buffer, ContentType='image/jpeg')

    return {'statusCode': 200, 'body': json.dumps('Image processed successfully!')}

Step 5: Set S3 in Lambda triggers
On the lambda function page, click on "Add triggers"
    Add triggers
    Select a source: S3
    Bucket: vcc.bucket
    Event types: PUT
    Add

Step 6: Create an API Gateway for upload and retrieval of images.
Go to API Gateway in the AWS Management Console.
Select "REST API" and click on "Build"
    Create REST API
    API details: New API
    API name: Image API
    API endpoint type: Regional
    Create API

For uploading images:
    Create resource
    Resource details
    Resource path: /
    Resource name: upload

    Create method
    Method details
    Method type: PUT
    Integration type: AWS service
    AWS Region: ap-south-1 (Same as region of the bucket)
    AWS service: Simple Storage (S3)
    HTTP method: PUT
    Action type: Use path override
    Path override - optional: rest/resource/upload
    Execution role: Set up the method execution with the necessary request and response transformations.
    Credential cache: Do not add caller credentials to cache key.
    Default timeout: 29 seconds.

For retrieval images:
    Create resource
    Resource details
    Resource path: /
    Resource name: reterieval

    Create method
    Method details
    Method type: GET
    Integration type: AWS service
    AWS Region: ap-south-1 (Same as region of the bucket)
    AWS service: Simple Storage (S3)
    HTTP method: GET
    Action type: Use path override
    Path override - optional: rest/resource/upload
    Execution role: Set up the method execution with the necessary request and response transformations.
    Credential cache: Do not add caller credentials to cache key.
    Default timeout: 29 seconds.

Step 8: Deploy API
Deploy API
    Stage: *New stage*
    Stage name: image
    Click on Deploy

Now test it by uploading an image to your S3 bucket using the API Gateway's
URL and retrieving processed images the same way.

Q2. Design a serverless email subscription service that captures user email addresses through an API Gateway, triggers a Lambda function to store them in an Amazon DynamoDB table, and sends a confirmation email using Amazon Simple Email Service (SES). Test the system's functionality using Postman.

Step 1: Setup Amazon DynamoDB
Go to DyanamoDB in the AWS Management Console.
Create table
    Table details
    Table name: emailSubscriptions
    Partition key: Email    String
    and leave other to default

Step 2: Create a Simple Email Service (SES)
Go to Amazon Simple Email Service (SES) in the AWS Management Console.
    Enter your email: user@gmail.com
    Domain: user@gmail.com
    Create

Navigate to Configurations in left sidebar and click on "Configuration Set".
Create set
    Configuration set name: EmailSubscriptionConfig

Step 3: Create a Lamda Function.
Go to the Lambda in the AWS Management Console.
To process the image this function will be triggered
Create a Lambda function
    Create function
    Author from scratch
	Function name: saveEmailSubscription
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Create a new role with basic Lambda permissions
	Advance settings -> Enable function URL
    Create function

Lambda Function Code:
import json
import boto3

dynamoDB = boto3.resource('dynamodb').Table('emailSubscriptions')
ses = boto3.client('ses')

def lambda_handler(event, context):
    email = json.loads(event['body'])['email']
    dynamoDB.put_item(Item={'Email': email})
    
    ses.send_email(
        Source='user@gmail.com',
        Destination={'ToAddresses': [email]},
        Message={
            'Subject': {'Data': 'Subscription Confirmation'},
            'Body': {'Text': {'Data': 'Thank you for subscribing!'}}
        }
    )

    return {
        'statusCode': 200,
        'body': json.dumps('Email subscribed successfully')
    }

Deploy the code

Step 4: Create an API Gateway.
Go to API Gateway in the AWS Management Console.
Select "REST API" and click on "Build"
    Create REST API
    API details: New API
    API name: Email API
    API endpoint type: Regional
    Create API

    Create resource
    Resource details
    Resource path: /
    Resource name: subscribe

    Create method
    Method details
    Method type: POST
    Integration type: Lambda function
    AWS Region: ap-south-1 (Same as region of the bucket)
    Choose the lambda function: EmailSubscriptionFunction
    Default timeout: 29 seconds.

    Deploy API
    Stage: *New stage*
    Stage name: email
    Click on Deploy

Step 5: Test with Postman
Open Postman.
    Create a new request to test the API.
    Set the method to POST and paste the API endpoint URL you got when from deployed API Gateway.

Create a new POST request.
Test Case:

{
"body": "{\"email\": \"user@gmail.com\"}"
}

Q3. Design a serverless data aggregation and reporting system using AWS Lambda and API Gateway. Users should be able to submit data via the API Gateway, and a Lambda function should aggregate and store the data in Amazon Redshift or a similar data store. Users can then retrieve reports via the API Gateway.

Step 1: Setup Amazon DynamoDB
Go to DyanamoDB in the AWS Management Console.
Create table
    Table details
    Table name: dataTable
    Partition key: ID    Number
    and leave other to default

Select the dataTable and click on Explore table items
Select Create item
    Add new attribute
    Price   Number
    Tax     Number
    Total   Number

Step 2: Create Lambda Functions for Data Aggeration.
Go to AWS Lambda in the AWS Management Console.
Create a Lambda function
    Create function
    Author from scratch
	Function name: data_function
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Create a new role with basic Lambda permissions
	Advance settings -> Enable function URL
    Create function

Lambda function code:
    import boto3
    import json
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('dataTable')

    def lambda_handler(event, context):
        try:
            # Extract data from the incoming event
            data = json.loads(event['body'])
            item_id = data.get('ID')
            price = data.get('Price')
            tax = data.get('Tax')

            # Calculate amount
            amount = price + tax

            # Insert data into DynamoDB
            response = table.put_item(
                Item={
                    'id': ID,
                    'price': Price,
                    'tax': Tax,
                    'amount': Amount
                }
            )

            return {
                'statusCode': 200,
                'body': 'Data inserted into DynamoDB successfully'
            }

        except Exception as e:
            return {
                'statusCode': 500,
                'body': f'Error inserting data into DynamoDB: {str(e)}'
            }

Test case:
test case {
  "body": "{ \"id\": 123, \"price\": 100, \"quantity\": 10 }"
}

Step 3: Create a Lambda function for data retriveal
Go to AWS Lambda in the AWS Management Console.
Create a Lambda function
    Create function
    Author from scratch
	Function name: retrive_function
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Create a new role with basic Lambda permissions
	Advance settings -> Enable function URL
    Create function

Lambda function code:
    import boto3
    import json
    from decimal import Decimal

    # Initialize a DynamoDB resource
    dynamodb = boto3.resource('dynamodb')

    # Reference to your DynamoDB table
    table = dynamodb.Table('dataTable')

    def lambda_handler(event, context):
        try:
            # Fetch all items from the DynamoDB table
            response = table.scan()

            items = response.get('Items', [])

            # Convert Decimal values to float
            for item in items:
                for key, value in item.items():
                    if isinstance(value, Decimal):
                        item[key] = float(value)

            return {
                'statusCode': 200,
                'body': json.dumps(items)
            }
        except Exception as e:
            return {
                'statusCode': 500,
                'body': json.dumps(f'An error occurred: {str(e)}')}

Test case 
{}

Step 3: Create an API Gateway
Go to API Gateway in the AWS Management Console.
Select "REST API or HTTP API" and click on "Build"
    Create HTTP API 
    API details: New API
    API name: Data API
    API endpoint type: Regional
    Create API

    Create resource
    Resource details
    Resource path: /
    Resource name: sumbitData

    Create method
    Method details
    Method type: POST
    Integration type: Lambda function
    AWS Region: ap-south-1
    Choose the lambda function: data_function
    Default timeout: 29 seconds.

    Deploy API
    Stage: *New stage*
    Stage name: submit
    Click on Deploy

    Create resource
    Resource details
    Resource path: /
    Resource name: getReport

    Create method
    Method details
    Method type: GET
    Integration type: Lambda function
    AWS Region: ap-south-1
    Choose the lambda function: get_function
    Default timeout: 29 seconds.


    Deploy API
    Stage: *New stage*
    Stage name: get
    Click on Deploy

Q4. Build a serverless URL shortening service using AWS Lambda and API Gateway. When users submit a long URL via the API Gateway, a Lambda function should generate a short URL and redirect users to the original URL when they access the short URL. Test the functionality using Postman.

Q5. You are tasked with designing a VPC in AWS, and you've chosen the CIDR block 172.31.0.0/16 for your private subnet. Calculate and provide the total number of available IP addresses in this CIDR block for your private subnet.

/16 network has 16 bits for the network and 16 bits for host addresses.
2^(32-16) = 2^16 = 65536

This means that the actual number of usable IP addresses will be 65536 - 5 = 65531.


Q6. In your AWS VPC, you've allocated the CIDR block 192.168.0.0/20. Calculate the number of available IP addresses for both the public and private subnets if you plan to divide this CIDR into two equal-sized subnets.

/20 CIDR block contains 232−20=212232−20=212
IP addresses, which is 4,096
subnet mask—going from /20 to /21

/21 subnet has 232−21=211232−21=211 IP addresses, which is 2,048
2,048 - 5 = 2,043.

Both the public and private subnets will have the same number of usable IP addresses since they are of equal size. Thus, each subnet will have 2,043 usable IP addresses.

Q7. You are designing a network infrastructure for a company that requires three subnets for different purposes, each with unique CIDR blocks. Calculate and provide the total number of available IP addresses in each of the following CIDR blocks: 10.0.0.0/24, 192.168.1.0/28, and 172.16.0.0/23.

The number of available IP addresses in a subnet can be calculated using the
formula:
Number of IP addresses=2(32−subnet mask)Number of IP addresses=2(32−subnet mask)

The total number of available IP addresses for each of the CIDR blocks is as follows:
1. 10.0.0.0/24: The subnet mask is 24, so the calculation is
2(32−24)−2=28−2=256−22(32−24)−2=28−2=256−2.
2. 192.168.1.0/28: The subnet mask is 28, so the calculation is
2(32−28)−2=24−2=16−22(32−28)−2=24−2=16−2.
3. 172.16.0.0/23: The subnet mask is 23, so the calculation is
2(32−23)−2=29−2=512−22(32−23)−2=29−2=512−2.

For 10.0.0.0/24, there are 254 available IP addresses.
For 192.168.1.0/28, there are 14 available IP addresses.
For 172.16.0.0/23, there are 510 available IP addresses.


Q8. You've been allocated a block of IP addresses in the format 203.0.113.0/24. Calculate and provide the total number of available IP addresses in this block. If you need to expand the range to accommodate more devices, what is the new CIDR block if you add an additional /25 subnet?

The original block 203.0.113.0/24 has 254 available IP addresses.

If you need to expand the range to accommodate more devices and add an additional /25 subnet, the new CIDR block would be a /23 to double the address space. This new /23 block would provide 510 available IP addresses. However, it's important to clarify that expanding a subnet range typically involves planning and coordination with your ISP or internal network team, as it's not as simple as changing the subnet mask; the routing and address allocation must also support this change.

# For the original CIDR block 203.0.113.0/24
original_subnet_mask = 24

# Calculate the total number of available IP addresses
total_ips_original = (2**(32 - original_subnet_mask)) - 2
# When adding an additional /25 subnet, the new range will be a /23 subnet, because a /23 is one bit less than /24, effectively doubling the address space of /24.
new_subnet_mask = original_subnet_mask - 1

# Calculate the total number of available IP addresses in the new range
total_ips_new_range = (2**(32 - new_subnet_mask)) – 2


Q9. You have multiple smaller CIDR blocks in the format 192.168.1.0/26, 192.168.1.64/26, and 192.168.1.128/26. Calculate and provide the total number of available IP addresses when these CIDR blocks are combined into a supernet.

When the smaller CIDR blocks 192.168.1.0/26, 192.168.1.64/26, and 192.168.1.128/26 are combined into a supernet, they fit into a larger /24 block because they are contiguous and part of the 192.168.1.0/24 network. The total number of available IP addresses in this supernet would be 254.
# For supernetting, the combined CIDR block would need to encompass all the smaller blocks.
# The smaller blocks given are contiguous and fit into a /24 when combined.
# So the supernet block will be 192.168.1.0/24.
supernet_mask = 24  # Supernet mask for a /24 block
# Calculate the total number of available IP addresses for the supernet block
total_ips_supernet = (2**(32 - supernet_mask)) – 2


Q10. Suppose you've set up a budget named "Monthly AWS Costs" with a budgeted amount of $1,000. You've chosen to receive email notifications when 80% of the budget is consumed. If your AWS costs exceed $800 in a month, you will receive an email notification to alert you of the increased spending.

Step 1: Login through AWS Admin.
Step 2: Search for Billing and Cost Management in AWS Console.
Step 3: Navigate to Budgets on the left sidebar.
Step 4: Click on Create a budget.
            Budget setup: Customize (advanced)
            Budget types: Cost budget - Recommended
            Next -> Set your budget
            Budget name: Monthly AWS Costs
            Set budget amount
            Period: Monthly
            Budget renewal type: Recurring budget
            Start month: Nov    2023
            Budgeting method: Fixed
            Enter your budgeted amount ($): 1000
            Next -> Configure alerts
            Add an alert threshold
            Alert #1
            Threshold: 80   % of budget amount
            Notification preferences
            Email recipients: user@gmail.com
            Next -> Attach actions: None
            Next -> Review
            Create a budget

Q11. You've implemented the order processing system, but your e-commerce website has experienced a sudden surge in orders. Explain how you can configure the components (Lambda, SQS, DynamoDB) to handle increased workload and ensure the system remains scalable and reliable.

AWS Lambda: 
    Increase Concurrent Executions: In the Lambda function configuration, adjust the "Concurrency" settings to allow more simultaneous executions. This enables Lambda to handle more requests concurrently.

    Optimize Function Code: Optimize your Lambda function code for performance and efficiency. Ensure that the code is well-structured and makes use of best practices to minimize execution time.

Amazon SQS: 
    Queue Configuration: Configure your SQS queues to handle increased message throughput. You can adjust the "Message Retention Period" and "Maximum Message Size" based on your specific requirements.

    Scaling SQS: SQS is designed to automatically scale with the load. Ensure that you are using the standard queue type, which can handle a virtually unlimited number of messages per second. You can also leverage the auto-scaling capabilities of SQS to dynamically adjust the number of concurrent messages based on the workload.

Amazon DynamoDB: 
    Provisioned Throughput: DynamoDB allows you to provision throughput capacity for your tables. Adjust the read and write capacity units based on the increased workload. DynamoDB also has on-demand capacity, which automatically scales based on the actual request volume.

    DynamoDB Accelerator (DAX): Consider using DynamoDB Accelerator (DAX) for caching frequently accessed data. This can reduce the load on DynamoDB and improve response times.

    Partition Key Design: Ensure your DynamoDB table has an effective partition key design to distribute the data evenly across partitions. This prevents hot partitions and ensures a more balanced workload.

Auto Scaling: Leverage AWS Auto Scaling to automatically adjust the number of resources (e.g., EC2 instances, Lambda functions) based on demand. This helps in scaling up during peaks and scaling down during periods of low demand.

Monitoring and Logging:
    Implement robust monitoring using AWS CloudWatch to track the performance of Lambda functions, SQS queues, and DynamoDB tables. Set up alarms to be notified of any performance issues.

    Use AWS X-Ray for tracing and debugging distributed applications. This can help identify bottlenecks and areas for optimization.

Backup and Recovery: Implement backup and recovery mechanisms for critical data stored in DynamoDB. Ensure that you have a strategy for data durability and resilience against failures.

By configuring these components with scalability and reliability in mind, you can ensure that your order processing system remains responsive and can handle sudden increases in workload efficiently. Regular testing and monitoring are crucial to identifying potential issues and optimizing the system further.

Q12. With above questions - Discuss error handling and retry strategies for the order processing system. How can you ensure that failed processing attempts are retried and that error notifications are sent to a designated team for further investigation?

Implementing effective error handling and retry strategies is crucial for maintaining the reliability and robustness of an order processing system. Here are some considerations and strategies:

Error Handling in Lambda Functions:
    Use Try-Catch Blocks: Wrap critical sections of your Lambda function code with try-catch blocks to catch and handle exceptions.
    Custom Error Messages: Provide clear and informative error messages to aid in troubleshooting.

Automatic Retries with SQS:
    Dead Letter Queues (DLQ): Configure a Dead Letter Queue for your SQS queues. Messages that repeatedly fail processing attempts are moved to the DLQ, allowing you to analyze and address the root cause.
    Retry Policy: Implement a retry policy for messages in the SQS queue. Configure the queue to automatically retry processing failed messages after a certain delay. Adjust the number of retries and delays based on the nature of the errors.
Exponential Backoff: Retry with Increasing Delays: Implement an exponential backoff strategy for retries. This means that if a processing attempt fails, the system waits for an increasing amount of time before retrying. This helps prevent overwhelming downstream services with repeated rapid retries.

CloudWatch Alarms and Metrics:
    Set Up Alarms: Configure CloudWatch alarms to monitor key metrics, such as the number of failed Lambda invocations or the size of the DLQ. Set up alarms to notify the team when error rates exceed a certain threshold.
    Custom Metrics: Instrument your Lambda functions and other components to emit custom metrics. This can help you track specific error conditions and performance metrics.

Logging and Auditing:
    CloudWatch Logs: Ensure that your Lambda functions log relevant information, including errors, to CloudWatch Logs. This provides a valuable resource for debugging and analyzing issues.
    Audit Trail: Maintain an audit trail of processing attempts and outcomes. This can assist in identifying patterns and trends in errors.

Notification Systems:
    SNS (Simple Notification Service): Integrate AWS SNS to send notifications to a designated team or individuals when critical errors occur. SNS can trigger notifications through various channels, including email, SMS, or even triggering other Lambda functions.
    Integration with Incident Management Tools: Consider integrating error notifications with incident management tools like PagerDuty or OpsGenie for a more comprehensive response to critical issues.

Monitoring and Continuous Improvement:
    Automated Monitoring: Implement automated monitoring of your entire order processing pipeline to detect anomalies and errors promptly.
    Regular Review and Improvement: Conduct regular reviews of error logs, metrics, and notifications. Use the insights gained to continually refine your error handling and retry strategies.

By combining these strategies, you can build a resilient order processing system that not only automatically retries failed processing attempts but also notifies the appropriate teams for further investigation and resolution. Regularly reviewing and refining these strategies based on real-world observations will contribute to the overall reliability of the system.

Q13. You're responsible for managing the budget for the order processing system. Describe how you can set up cost monitoring, budget alerts, and usage reports to keep track of the expenses associated with Lambda, SQS, and DynamoDB. What are some best practices for staying within budget while maintaining system performance?

Steps
Managing the budget for an order processing system on AWS involves setting up effective cost monitoring, budget alerts, and usage reports. Here are steps and best practices for cost management:

1. Cost Monitoring:
AWS Cost Explorer: Utilize AWS Cost Explorer to visualize, understand, and analyze your AWS costs. This service provides an interactive interface to explore your historical spending patterns and identify cost trends.
Cost Allocation Tags: Tag your resources with meaningful cost allocation tags. This enables you to break down costs by different dimensions, such as environment (e.g., production, development), department, or project.

2. Budgets and Alerts:
Set Budgets: Establish AWS Budgets to define the maximum amount you are willing to spend on specific AWS services or resources. Budgets can be set monthly, quarterly, or annually.
Budget Actions: Configure budget actions to trigger notifications or automated responses when certain cost thresholds are reached. For example, you can receive an alert when you are projected to exceed 80% of your budget.

3. Usage Reports:
AWS Cost and Usage Reports: Enable and configure AWS Cost and Usage Reports to receive detailed information about your AWS usage and associated costs. This can be stored in an Amazon S3 bucket for further analysis.
Granular Reporting: Use the detailed reports to understand the costs associated with individual services, such as Lambda, SQS, and DynamoDB. Identify areas where costs can be optimized.

4. Lambda Function Configuration:
Memory and Timeout Settings: Optimize the memory and timeout settings for your Lambda functions. Over-allocating resources can increase costs. Monitor function performance and adjust these settings based on actual requirements.
Cold Starts: Understand and mitigate the impact of cold starts on Lambda function performance. Consider provisioned concurrency for critical functions to minimize cold start times.

5. SQS Configuration:
Message Retention Period: Adjust the message retention period in SQS based on your requirements. Shorter retention periods can reduce storage costs.
Visibility Timeout: Optimize the visibility timeout to prevent unnecessary reprocessing of messages.

6. DynamoDB Configuration:
Provisioned Throughput: Regularly review and adjust provisioned throughput for your DynamoDB tables based on actual usage patterns. Use on-demand capacity if the workload is variable.
Index Usage: Be mindful of the indexes you create and their impact on read and write costs. Unused or unnecessary indexes can be a source of additional costs.

7. Monitoring and Alerts:
CloudWatch Alarms: Set up CloudWatch Alarms to monitor key performance metrics for Lambda, SQS, and DynamoDB. This includes metrics related to errors, latency, and throughput.
Automated Actions: Configure automated actions, such as scaling policies, based on CloudWatch Alarms to dynamically adjust resources in response to changing workloads.

8. Regular Cost Reviews:
Regularly Review Costs: Conduct regular reviews of your AWS costs and usage reports. This helps you identify areas for optimization and adjust your budget accordingly.

9. Cost Optimization Best Practices:
Reserved Instances and Savings Plans: Consider purchasing Reserved Instances or Savings Plans for predictable workloads. This can provide significant cost savings compared to on-demand pricing.
Use of Spot Instances: For non-critical or fault-tolerant workloads, consider using Spot Instances to take advantage of cost savings.

By implementing these practices, you can effectively monitor and manage the costs associated with Lambda, SQS, and DynamoDB in your order processing system. Regularly reviewing and optimizing your resources based on actual usage patterns will help you stay within budget while maintaining system performance.

Q14. Describe how to set up a DynamoDB trigger that invokes a Lambda function when changes occur in a specific DynamoDB table. Explain the different trigger types, such as stream-based and batch-based, and provide an example use case for each.

Steps
Setting up a DynamoDB trigger to invoke a Lambda function when changes occur in a DynamoDB table involves using DynamoDB Streams in combination with AWS Lambda. Here's a step-by-step guide to setting it up, followed by an explanation of different trigger types and example use cases for each.

# Setting Up a DynamoDB Trigger with Lambda
1. Enable DynamoDB Streams on Your Table:
    - Go to the AWS Management Console, navigate to the DynamoDB section, and select your table.
    - In the table settings, enable DynamoDB Streams. You can choose the type of information the stream captures (e.g., keys only, new image, old image, or both new and old images).
2. Create a Lambda Function:
    - Go to the AWS Lambda console and create a new function.
    - Choose the runtime environment and provide the code for your Lambda function. This function will process the data from the DynamoDB Stream.
3. Set the DynamoDB Stream as a Trigger:
    - In the Lambda function configuration, add a trigger. Select DynamoDB from the list of available triggers.
    - Configure the trigger by selecting the DynamoDB table and stream. You also need to define the batch size, which is the number of records the Lambda function will process at once.
4. Deploy and Test:
    - Deploy your Lambda function.
    - Perform operations (create, update, delete) on your DynamoDB table to generate stream records and test if the Lambda _function is triggered as expected.

# Trigger Types
1. Stream-based Trigger:
    - Description: This trigger is event-driven. When a change (insert, modify, delete) occurs in a DynamoDB table, the corresponding stream record is generated and the Lambda function is invoked in near real-time.
    - Use Case Example: A real-time dashboard that reflects changes in data. For instance, if you have a table storing inventory data, a Lambda function can update a dashboard in real-time whenever inventory levels change.

2. Batch-based Trigger (Poll-based):
    - Description: In this scenario, changes are not processed in real-time. Instead, the Lambda function polls the DynamoDB Stream at regular intervals and processes records in batches. This is a built-in feature of DynamoDB and Lambda integration, where Lambda polls the stream and invokes your function synchronously when there are records to process.
    - Use Case Example: A batch processing system for data aggregation. Imagine a scenario where you're collecting user activity data in a DynamoDB table. A Lambda function can be configured to run at regular intervals, aggregating this data and updating a separate analytics table or system.

Q15. Write an AWS Lambda function in Python to process incoming JSON files and store them in an Amazon DynamoDB table. Provide a sample JSON file for testing.

Step 1: Setup Amazon DynamoDB
Go to DyanamoDB in the AWS Management Console.
Create table
    Table details
    Table name: jsonfile
    Partition key: jsonid    String
    and leave other to default

Step 2: Create an IAM Role for Lambda
Go to IAM in the AWS Management Console.
AWS Lambda will require permissions to access S3 and other AWS services.
Navigate on "Roles" from the left sidebar, and then click on "Create role".
    Trusted entity type: AWS service
    Use case: Lambda
    Next -> Add permissions
    Permissions policies
    Permissions policies: AmazonDynamoDBFullAccess and AWSLambdaExecute
    Next -> Name, review, and create
    Role details
    Role name: jsonrole
    Step 1 and Step 2 as it is
    Step 3: Add tags (add tags if needed)
    Create role

Step 3: Create a Lamda Function.
Go to the Lambda in the AWS Management Console.
To process the image this function will be triggered
Create a Lambda function
    Create function
    Author from scratch
	Function name: json_function
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Use an existing role: jsonrole
	Advance settings -> Enable function URL
    Create function


Lamda function code:
    import json
    import boto3

    def lambda_handler(event, context):
        records = event.get('Records', [])
        dynamodb_table = boto3.resource('dynamodb').Table('jsonfile')

        for record in records:
            json_data = json.loads(record['body'])
            item = {
                'id': json_data.get('id'),
                'name': json_data.get('name'),
                'value': json_data.get('value')
            }

            print(item)
            dynamodb_table.put_item(Item=item)

        return {'statusCode': 200, 'body': 'Data stored in DynamoDB successfully!'}

Test case:
    {
    "Records": [
        {
        "body": "{\"id\": \"5\", \"name\": \"Test Item\", \"value\": 789}"
        }
    ]
    }

Q16. Set up an S3 bucket for storing incoming JSON files. Configure the bucket with appropriate access policies and versioning.Configure an S3 event to trigger the Lambda function whenever a new JSON file is uploaded. Create an SNS topic to notify stakeholders when file processing is complete. Set up subscriptions for relevant email addresses.

Step 1: Create an S3 Bucket.
Go to S3 in the AWS Management Console.
    Create bucket
    Bucket name: json.bucket
    AWS Region: Asia Pacific(Mumbai) ap-south-1 (select the AWS Region you want to create your bucket in)
    Bucket Versioning: Enable
    disable Block all public access
    keep rest of the settings default
    Create bucket

Step 2: Create a SNS.
Go to the Simple Notification Service in the AWS Management Console.
Create a SNS
    Create topic
    Details
    Type: Standard
    Name: jsonemail
    Create
After creating topic -> Create subscription
    Topic ARN: jsonemail
    Protocol: Email
    Endpoint: user@gmail.com
    Create

Note down the arn
	
Step 3: Go to S3 Bucket
Go to S3 in the AWS Management Console.
Select the bucket you created: vcc.bucket
    properties → Event notifications → Create event notification
    Name: JSONMail
    Events: Choose "All objects create events."
    Send to: Choose "Lambda Function": jsonfunction

Step 4: Create a Lamda Function.
Go to the Lambda in the AWS Management Console.
Create a Lambda function
    Create function
    Author from scratch
	Function name: json_function
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Use an existing role: jsonrole
	Advance settings -> Enable function URL
    Create function

Lambda function code:
import json
import boto3


ses = boto3.client('ses')
sns = boto3.client('sns')

def lambda_handler(event, context):
    try:
        email = 'user@gmail.com'


        # Send confirmation email using SES
        send_confirmation_email(email)

        # Publish to SNS
        sns.publish(
            TopicArn='arn:aws:sns:ap-south-1:540280354050:q16',  # Replace with your SNS topic ARN
            Message='File processing is complete!',
            Subject='File Processing Notification'
        )

        return {
            'statusCode': 200,
            'body': json.dumps('Email subscribed successfully')
        }
    except Exception as e:
        print(f"Error: {e}")
        return {
            'statusCode': 500,
            'body': json.dumps('Error subscribing email')
        }

def send_confirmation_email(email):
    # Replace with the email you verified in SES
    sender_email = 'user@gmail.com'
    subject = 'Subscription Confirmation'
    body_text = f'Thank you for subscribing to our newsletter!'

    ses.send_email(
        Source=sender_email,
        Destination={'ToAddresses': [email]},
        Message={
            'Subject': {'Data': subject},
            'Body': {'Text': {'Data': body_text}}
        }
    )

    print(f"Confirmation email sent to {email}")

Q17. You are tasked with designing a text processing system for your company. The system needs to process incoming text data, perform analysis, and store the results in a scalable and efficient manner.

To achieve this, you will leverage various AWS services. Requirements:
1. AWS Lambda Functions: Create AWS Lambda functions for text processing and analysis. These functions should take incoming text data, perform custom processing or analysis, and store the results.
2. Amazon S3: Set up an Amazon S3 bucket where incoming text files are stored. Configure event triggers to invoke the Lambda functions whenever new files are uploaded.
3. Amazon SQS: Implement an SQS queue to manage text processing requests, ensuring reliable processing and scaling when the volume of text data increases.
4. Amazon SNS: Establish SNS topics to notify stakeholders when text processing is complete, delivering notifications to relevant email addresses or other endpoints.
5. Amazon DynamoDB: Create a DynamoDB table to store metadata and analysis results from processed text data, allowing for efficient retrieval and query operations.
6. Security: Implement proper security measures, including IAM roles and permissions, to safeguard data and processing resources. Ensure that access to resources is restricted appropriately.
7. Monitoring and Logging: Set up AWS CloudWatch for monitoring and logging to maintain visibility into system operations. Configure alarms and log retention policies for effective monitoring.

Step 1: Setup Amazon DynamoDB
Go to DyanamoDB in the AWS Management Console.
Create table
    Table details
    Table name: textProcessingTable
    Partition key: id    String
    and leave other to default

Step 2: Create an S3 Bucket.
Go to S3 in the AWS Management Console.
    Create bucket
    Bucket name: text-processing-bucket
    AWS Region: Asia Pacific(Mumbai) ap-south-1 (select the AWS Region you want to create your bucket in)
    disable Block all public access
    keep rest of the settings default
    Create bucket

Step 3: Create a SQS Queue
Go to Simple Queue Service in the AWS Management Console.
Click on Create queue
    Type: Standard
    Queue name: textProcessingQueue.
    Configure other settings as needed.
    Click on Create queue.

Step 4: Create a SNS.
Go to the Simple Notification Service in the AWS Management Console.
Create a SNS
    Create topic
    Details
    Type: Standard
    Name: textProcessingTopic
    Create
After creating topic -> Create subscription
    Topic ARN: textProcessingTopic
    Protocol: Email
    Endpoint: user@gmail.com
    Create

Note the Topic ARN

Step 5: Create a Lamda Function.
Go to the Lambda in the AWS Management Console.
Create a Lambda function
    Create function
    Author from scratch
	Function name: textProcessingFunction
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Use an existing role: textprocessrole 
    (Attach neccessary permission like AmazonS3FullAccess, AmazonSQSFullAccess, AmazonSNSFullAccess and AmazonDynamoDBFullAccess)
	Advance settings -> Enable function URL
    Create function

Lambda function code:
    import json
    import boto3
    import urllib.parse

    dynamoDB = boto3.resource('dynamodb').Table('textProcessingTable')
    ses = boto3.client('ses')
    sns = boto3.client('sns')
    s3 = boto3.client('s3')

    def lambda_handler(event, context):
        try:
            # Retrieve S3 bucket and object information from the S3 event
            bucket = event['Records'][0]['s3']['bucket']['name']
            key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
            
            # Retrieve the text data from the uploaded file
            text_data = get_text_from_s3(bucket, key)
            print(text_data)

            # Perform basic analysis (word count)
            words = text_data.split()
            word_count = len(words)
            print(word_count)

            # Store results in DynamoDB
            response = dynamoDB.put_item(
                Item={
                    'id': '1',  # You might want to generate a unique ID
                    'count': word_count
                }
            )
            email = 'user@gmail.com'

            # Send confirmation email using SES
            send_confirmation_email(email)

            # Publish to SNS
            sns.publish(
                TopicArn='arn:aws:sns:ap-south-1:540280354050:q17',  # Replace with your SNS topic ARN
                Message=f'File processing is complete! Words: {word_count}',
                Subject='File Processing Notification'
            )

            return {
                'statusCode': 200,
                'body': json.dumps('Text analysis complete and stored in DynamoDB')
            }
        except Exception as e:
            print(f"Error: {e}")
            return {
                'statusCode': 500,
                'body': json.dumps('Error processing text data')
            }

    def send_confirmation_email(email):
        # Replace with the email you verified in SES
        sender_email = 'user@gmail.com'
        subject = 'Subscription Confirmation'
        body_text = f'Thank you for subscribing to our newsletter!'

        ses.send_email(
            Source=sender_email,
            Destination={'ToAddresses': [email]},
            Message={
                'Subject': {'Data': subject},
                'Body': {'Text': {'Data': body_text}}
            }
        )

        print(f"Confirmation email sent to {email}")

    def get_text_from_s3(bucket, key):
        # Retrieve the content of the S3 object (text file)
        response = s3.get_object(Bucket=bucket, Key=key)
        text_data = response['Body'].read().decode('utf-8')
        return text_data
    
Test Case    
    {
        "text": "Exam Exam Exam"
    }

Step 6: For the S3 bucket:
Navigate to the S3 bucket you created. 
Go to the "Properties" section and find the "Event notifications" setting.
Add a new notification, select the event type (e.g., "All object create events"), and select the Lambda function as the destination.

Step 7: IAM Roles and Permissions for Lambda Function:
Lambda Service Role:
This role is assumed by the Lambda service when it executes your function.
It should have permissions to execute Lambda functions and access CloudWatch Logs.
Attach the AWSLambdaBasicExecutionRole policy for basic Lambda execution.
Attach the AWSLambdaRole policy for writing logs to CloudWatch.

Step 8: Monitoring and logging 
  Go to lambda console , monitoring and operations tools , additional edit monitoring  and enable everything.

Q18. You are tasked with designing a secure and efficient system for processing sensitive JSON files in a company. The company receives a constant stream of JSON files that contain sensitive information. The files must be processed, stored securely, and analyzed. The following requirements have been defined:

1. AWS Lambda: Develop AWS Lambda functions for JSON processing and analysis. These functions will securely store the JSON files in an Amazon DynamoDB table with encryption enabled.
2. Amazon S3: Set up an S3 bucket with versioning enabled where incoming JSON files are securely stored. Configure event triggers to invoke Lambda functions whenever new files are uploaded.
3. Amazon SQS: Implement an SQS queue to manage file processing requests, ensuring reliable processing and scaling when the volume of files increases. Ensure that the queue is encrypted and access is restricted.
4. Amazon SNS: Establish SNS topics to notify stakeholders when file processing is complete, delivering notifications to relevant email addresses. Ensure that SNS topics are encrypted in transit.
5. Amazon DynamoDB: Create a DynamoDB table to store metadata and analysis results from processed JSON files, allowing for efficient retrieval and query. Encrypt the DynamoDB table at rest.
6. Security: Implement proper security measures, including IAM roles and permissions, to safeguard data and processing resources. Enable AWS Key Management Service (KMS) encryption for Lambda, S3, SQS, SNS, and DynamoDB.
7. Monitoring and Logging: Set up CloudWatch for monitoring and logging to maintain visibility into system operations. Enable CloudTrail for auditing API actions. Create CloudWatch Alarms for important metrics.

Step 1: Setup Amazon DynamoDB
Go to DyanamoDB in the AWS Management Console.
Create table
    Table details
    Table name: textProcessingTable
    Partition key: id    String
    and leave other to default

Step 2: Create an S3 Bucket.
Go to S3 in the AWS Management Console.
    Create bucket
    Bucket name: json-bucket
    AWS Region: Asia Pacific(Mumbai) ap-south-1 (select the AWS Region you want to create your bucket in)
    disable Block all public access
    keep rest of the settings default
    Create bucket

Step 3: Create a SQS Queue
Go to Simple Queue Service in the AWS Management Console.
Click on Create queue
    Type: Standard
    Queue name: fileProcessingQueue.
    Configure other settings as needed.
    Click on Create queue.

Step 4: Create a SNS.
Go to the Simple Notification Service in the AWS Management Console.
Create a SNS
    Create topic
    Details
    Type: Standard
    Name: fileProcessingTopic
    Create
After creating topic -> Create subscription
    Topic ARN: textProcessingTopic
    Protocol: Email
    Endpoint: user@gmail.com
    Create

Note the Topic ARN

Step 5: Create a Lamda Function.
Go to the Lambda in the AWS Management Console.
Create a Lambda function
    Create function
    Author from scratch
	Function name: jsonProcessingFunction
	Runtime: Python 3.11
	Architecture: x84_64
    Change default execution role ->
    Execution role: Use an existing role: jsonprocessrole 
    (Attach neccessary permission like AmazonS3FullAccess, AmazonSQSFullAccess, AmazonSNSFullAccess and AmazonDynamoDBFullAccess)
	Advance settings -> Enable function URL
    Create function

Lambda function code:
    import json
    import boto3
    import urllib.parse

    # Initialize AWS clients
    s3 = boto3.client('s3')
    dynamoDB = boto3.resource('dynamodb').Table('YourDynamoDBTableName')
    sns = boto3.client('sns')

    def lambda_handler(event, context):
        try:
            # Retrieve S3 bucket and object information from the S3 event
            bucket = event['Records'][0]['s3']['bucket']['name']
            key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
            
            # Retrieve the JSON data from the uploaded file
            json_data = get_json_from_s3(bucket, key)
            print(json_data)

            # Store sensitive JSON data securely in DynamoDB
            store_json_in_dynamodb(json_data)

            # Publish a notification to SNS
            publish_notification()

            return {
                'statusCode': 200,
                'body': json.dumps('JSON processing complete and stored in DynamoDB')
            }
        except Exception as e:
            print(f"Error: {e}")
            return {
                'statusCode': 500,
                'body': json.dumps('Error processing JSON data')
            }

    def get_json_from_s3(bucket, key):
        # Retrieve the content of the S3 object (JSON file)
        response = s3.get_object(Bucket=bucket, Key=key)
        json_data = json.loads(response['Body'].read().decode('utf-8'))
        return json_data

    def store_json_in_dynamodb(json_data):
        # Store the JSON data in DynamoDB
        response = dynamoDB.put_item(
            Item={
                'id': '1',  # You might want to generate a unique ID
                'data': json_data
            }
        )

    def publish_notification():
        # Publish a notification to SNS
        sns.publish(
            TopicArn='arn:aws:sns:your-region:your-account-id:YourSNSTopicName',  # Replace with your SNS topic ARN
            Message='File processing is complete! JSON data stored securely.',
            Subject='File Processing Notification'
        )

Step 6: Implement Security with IAM and AWS KMS
Navigate to the IAM service in the AWS Console.
Create new policies that define the precise permissions for Lambda, S3, SQS, SNS, and DynamoDB.
Attach these policies to the respective roles that your Lambda and other services will assume.
Navigate to the KMS service and set up keys for encrypting your resources. Assign aliases and define key administrative and usage permissions.

 Step 7: Monitoring and Logging with CloudWatch and CloudTrail
For CloudWatch:
    Navigate to the CloudWatch service.
    Go to "Logs" and ensure that Lambda and other services are configured to send logs to CloudWatch.
    Set up "Alarms" for metrics that are critical for your application's performance and security.
For CloudTrail:
    Navigate to the CloudTrail service.
    Click "Create trail."
    Enter a trail name, select an S3 bucket for storing logs, and configure other settings as necessary.
    Make sure to select "Write" and "Read/Write" events to monitor.

Q19. You are tasked with deploying a scalable and fault-tolerant web application for a media streaming company. The application includes both a back-end component. For this scenario, you will focus on using AWS Elastic Beanstalk to achieve the company's requirements:

1. Environment Configuration: Create an Elastic Beanstalk environment suitable for a Python and Flask-based backend application. Ensure the environment includes a web server.
2. High Availability: Set up the environment to be highly available by spanning it across multiple AWS Availability Zones. Describe the process and configurations needed for achieving fault tolerance.
3. Security and Scalability: Implement security best practices by configuring security groups and custom VPC (not in a default VPC). Additionally, set up autoscaling policies for the environment to handle traffic spikes effectively.
4. Monitoring and Metrics: Configure AWS CloudWatch to monitor the application's performance and set up alarms for critical metrics. Provide insights into which metrics are essential and the threshold values for the alarms.
5. Deployment Strategy: Define a manual deployment strategy, perform two deployments - in the first, deploy a basic Python Flask app, and in the second, make some changes redeploy to demonstrate deployment strategy.

Step 1: Open Amazon CloudShell and perform following commands
mkdir eb-flask
cd eb-flask
sudo vi application.py
sudo vi requirements.txt
python3 -m ensurepip --upgrade
pip3 install -r requirements.txt
eb init -p python-3.11 hello-world --region ap-south-1
eb create test-environment
eb use test-environment
eb status test-environment
eb deploy

Application.py
from flask import Flask

application = Flask(__name__)


@application.route("/")
def hello():
    """Hello word method."""
    return "Hello Calvine - This is a one step!"


# run the app.
if __name__ == "__main__":
    application.run()

# Esc(:wq)

Requirements.txt
Flask

Step 2: Go to Elastic Beanstalk in the AWS Management Console
Navigate to Configurations in Environment: test-environment section on left sidebar
Capacity → Placement ⇒ Select All
Apply changes

Q20. You are responsible for deploying a scalable and fault-tolerant web application for a media streaming company. The application includes both a back- end component based on Python and Flask. Your task is to design and implement an AWS Elastic Beanstalk environment that meets the company's requirements:

1. Environment Configuration: Create an Elastic Beanstalk environment suitable for a Python and Flask-based backend application. Ensure the environment includes a web server. Explain the necessary configurations for the environment.
2. High Availability: Set up the environment to be highly available by spanning it across multiple AWS Availability Zones. Describe the process and configurations needed for achieving fault tolerance. What considerations should be taken into account when deploying the application across AZs?
3. Security and Scalability: Implement security best practices by configuring security groups and a custom VPC (not in a default VPC). Additionally, set up autoscaling policies for the environment to handle traffic spikes effectively. Describe the security group rules and autoscaling configurations.
4. Monitoring and Metrics: Configure AWS CloudWatch to monitor the application's performance and set up alarms for critical metrics. Provide insights into which metrics are essential and the threshold values for the alarms. How would you set up custom CloudWatch alarms for your Flask application? 5. Deployment Strategy: Define a manual deployment strategy and perform two deployments:
- In the first deployment, deploy a basic Python Flask app to Elastic Beanstalk.
- In the second deployment, make some changes to the Flask app and demonstrate the deployment strategy. Explain the steps you take during the deployments.

Step 1: Environment Configuration for Python and Flask
Login to AWS Console:
    Go to AWS Management Console and sign in.

Step 2: Navigate to Elastic Beanstalk:
In the AWS Management Console, find and select "Elastic Beanstalk" under Services.

Step 3: Create New Application:
    Click on "Create New Application".
    Name your application (e.g., "MediaStreamingApp").

Step 4: Create New Environment:
    Inside your application, select "Create a new environment".
    Choose "Web server environment".

Step 5: Configure Environment:
    Platform: Select "Python" as the platform, and choose an appropriate version.
    Application Code: Upload your Flask application code (ZIP format).
    Environment Information: Provide a unique domain name or leave it to AWS to generate one.
        High Availability Across Availability Zones
        Environment Configuration:
            In the environment configuration, navigate to "Capacity".
            Select "High availability" with load balancing across multiple Availability Zones.
        
        Considerations for Multi-AZ Deployment:
            Ensure that your application is stateless.
            Use a centralized database service like Amazon RDS.
            Use Amazon S3 for static files and media storage.
        
        Security and Scalability
        Create a Custom VPC:
            Go to VPC Dashboard and create a new VPC.
            Define subnets in different Availability Zones.
        
        Security Groups:
            Create a security group for your web servers (allow ports 80 and 443 for HTTP/S).
            Create a security group for your database (allow the specific port only from the web server security group).
        
        Auto Scaling Configuration:
            In Elastic Beanstalk, navigate to "Configuration".
            Under "Scaling", configure the autoscaling policy (e.g., scale based on CPU utilization or network traffic).
            
        Monitoring and Metrics with AWS CloudWatch
        Configure CloudWatch:
            Go to the CloudWatch console.
            Enable detailed monitoring for your Elastic Beanstalk resources.
            
        Set Alarms:
            Create alarms for metrics like CPU Utilization, Network In/Out, and Error Rates.
            Define thresholds for triggering alarms (e.g., CPU usage above 70% for 5 minutes).
            
        Custom Metrics for Flask:
            Instrument your Flask application to send custom metrics to CloudWatch (use boto3 library for AWS integration).

Step 5:Deployment Strategy
First Deployment - Basic Flask App:
    Package your basic Flask app (app.py, requirements.txt, etc.) into a ZIP file.
    Use Elastic Beanstalk dashboard to upload and deploy this ZIP file.
    
Second Deployment - Updated Flask App:
    Make changes to your Flask app.
    Repackage and upload the new version of your app.
    Monitor the deployment status and health in the Elastic Beanstalk dashboard

Q21. You're tasked with setting up a scalable and secure Flask web application on AWS. The application will serve as an image uploading platform, allowing users to upload images, apply filters, and share them. Requirements:

1. Application Development: You have a Flask application codebase ready, including image uploading and filtering features.
2. Auto-Scaling: Implement an auto-scaling solution to handle varying user loads effectively. Configure scaling policies and triggers to adjust the number of application instances dynamically based on traffic fluctuations.
3. High Availability: Deploy the Flask application across multiple Availability Zones (AZs) to provide redundancy and high availability, minimizing downtime.
4. Security Measures: Implement security best practices, including securing communication over HTTPS, access controls, and safeguarding user- uploaded images. Ensure that application instances have the latest security patches.
5. Monitoring and Alerts: Set up comprehensive monitoring using AWS CloudWatch. Create meaningful alarms based on key performance metrics, such as response times, error rates, and resource utilization. Configure alerts to proactively respond to issues.
6. Amazon S3 Integration: Utilize Amazon S3 for storing user-uploaded images. Implement functionality in your Flask application to allow users to upload images to an S3 bucket and apply filters.

Step 1: Set Up AWS Environment
AWS Account & IAM User
    Create an AWS Account: Sign up for an AWS account at aws.amazon.com.
    Create IAM Users: In AWS Management Console, navigate to IAM service, create a new user with administrative permissions.

Step 2: Set Up Application Infrastructure
Elastic Beanstalk for Flask Application
    Create Elastic Beanstalk Application:
    Go to the Elastic Beanstalk console.
    Choose “Create New Application” and name it.
    Create a new environment (Web server environment).
    Choose Python as the platform and upload your Flask application code (ZIP file).
    Configure Environment:
        In your environment, go to “Configuration.”
        Under “Capacity” configure auto-scaling settings (min and max instances).
        Under “Load Balancer,” enable HTTPS by adding a listener on port 443.
    High Availability & Auto-Scaling
        Multi-AZ Deployment:
            In the Elastic Beanstalk environment configuration, under “Instances,” choose multiple availability zones.
        Auto-Scaling:
            Go to “Scaling” and set auto-scaling triggers based on CPU utilization or other metrics.

Step 3: Security Measures
HTTPS Configuration
    SSL Certificate:
        Request or import an SSL certificate using AWS Certificate Manager.
        Associate this certificate with your Elastic Beanstalk environment’s load balancer.

Security Groups & IAM Roles
    Security Groups:
        Configure security groups for your Elastic Beanstalk environment and RDS (if used) to only allow necessary traffic.
    
    IAM Roles:
        Ensure that the IAM role associated with your EB environment has the necessary permissions, especially for S3 access.

Step 4: S3 Integration for Image Storage
Create S3 Bucket
    Create S3 Bucket:
        Go to the S3 console and create a new bucket for storing images.
        Set up appropriate bucket policies and permissions for access.
        Integrate with Flask
    
    Modify Flask App:
        Ensure your Flask app uses AWS SDK (boto3) to interact with the S3 bucket.
        Implement image upload and retrieval functionality using S3 APIs.

Step 5: Monitoring & Alerts with CloudWatch
Set Up CloudWatch
    Monitoring:
        Use AWS CloudWatch to monitor your application.
        In the CloudWatch console, set up dashboards to track metrics like CPU utilization, request counts, etc.
    
    Alarms:
        Create alarms in CloudWatch for specific metrics (e.g., high error rates, high latency).
        Configure actions for these alarms (like notifications).

Step 6: Continuous Deployment (Optional)
CodeCommit, CodeBuild, CodePipeline:
    Use these services for continuous integration and deployment of your Flask app.
    Set up a pipeline that pulls from CodeCommit, builds with CodeBuild, and deploys to Elastic Beanstalk.

Step 7: Testing & Validation
Test Application:
    Access your application’s URL to ensure it’s running.
    Test the scaling by simulating high load.
    
Security Testing:
    Perform security checks (like penetration testing) to ensure your setup is secure.

Step 8: Regular Maintenance
Update Patches:
    Regularly update your application and environment for security patches.

Review Monitoring Data:
    Regularly check CloudWatch metrics and alarms.

Step 9: Documentation & Support
AWS Documentation:
    Refer to AWS Documentation for detailed guidance on each service.

AWS Support:
    For complex scenarios, consider contacting AWS Support.

Q22. Design and implement a robust and scalable infrastructure for a Flask application that accepts JSON files, processes them, and displays structured output. Ensure high availability, security, monitoring, and logging. Use Amazon S3 for storage. Steps:

1. Flask Application Development:
- Develop a Python Flask application that accepts JSON files, processes them, and displays the structured output.
- Test the Flask application locally to ensure it functions as expected.
2. Create an S3 Bucket:
- Log in to the AWS Management Console.
- Navigate to Amazon S3 and create a new S3 bucket to store incoming JSON files.
3. Set Up an Elastic Load Balancer (ELB):
- Create a new Elastic Load Balancer (ELB) to distribute incoming traffic among multiple instances.
4. Launch Configuration and Auto Scaling Group:
- Create a launch configuration specifying the Flask application and other configurations.
- Set up an Auto Scaling Group to launch instances based on the launch configuration.
5. Configure Autoscaling Policies:
- Create scaling policies for the Auto Scaling Group to dynamically adjust the number of instances based on traffic fluctuations. 6. High Availability:
- Configure the Auto Scaling Group to span across multiple Availability Zones (AZs) for redundancy and high availability.
7. Security Measures:
- Implement security groups and Network Access Control Lists (NACLs) to control inbound and outbound traffic.
- Ensure instances have the latest security patches using AWS Systems Manager.
8. Monitoring and Alerts:
- Set up CloudWatch alarms to monitor key performance metrics (CPU utilization, request latency, etc.).
- Configure alarms to trigger notifications when predefined thresholds are breached.
9. Logging and Tagging:
- Implement CloudWatch Logs to monitor application logs for troubleshooting.
- Apply resource tags to manage and organize AWS resources efficiently.
10. Testing and Verification:
- Upload sample JSON files to the S3 bucket and ensure the Flask application processes them correctly. - Generate traffic to the application to trigger autoscaling and verify its effectiveness.

Step 1: Flask Application Development
Develop Flask Application
    Environment Setup: Install Python and Flask on your local machine.
    Application Coding: Write a Flask application in Python that can accept and process JSON files. Use Flask's request object to handle incoming JSON files.
    Local Testing: Test the application locally by running flask run and use tools like Postman or curl to send JSON files to your application.

Prepare for Deployment
    Requirements File: Create a requirements.txt file with all necessary Python packages.
    Application Packaging: Package your application with a WSGI server like Gunicorn for production deployment.

Step 2: AWS S3 Bucket Setup
Create S3 Bucket
    AWS Management Console Login: Sign in to your AWS account.
    Navigate to S3: Go to the S3 service section.
    Create Bucket: Click "Create bucket", enter a unique name, select the region, and create the bucket.

Step 3: Elastic Load Balancer (ELB)
Set Up ELB
    Navigate to EC2 Dashboard: From the AWS Management Console, go to EC2.
    Load Balancers: Click on "Load Balancers" in the EC2 dashboard and then "Create Load Balancer".
    Choose Type: Select the appropriate type (e.g., Application Load Balancer).
    Configure ELB: Specify details like name, scheme, IP address type, and listeners (HTTP, HTTPS).
    Configure Security Settings and Groups as needed.

Step 4: Launch Configuration and Auto Scaling Group
Create Launch Configuration
    Navigate to Auto Scaling: Go to the "Auto Scaling Groups" section.
    Create Launch Configuration: Click on "Create Launch Configuration". Select an AMI, instance type, and configure instance details, including IAM roles and monitoring options.
    Specify User Data: Include scripts to install and start your Flask app upon instance launch.

Create Auto Scaling Group
    Choose Launch Configuration: Select the launch configuration you created.
    Set Up Network: Choose VPC and subnets.
    Configure Scaling Policies: Set parameters for scaling up and down based on desired conditions (e.g., CPU usage).

Step 5: Autoscaling Policies
Configure Scaling Policies
    Create Scaling Policies: In the Auto Scaling group, define policies for scaling out (adding instances) and scaling in (removing instances).

Step 6: High Availability
Multi-AZ Configuration
    Availability Zones: Ensure your Auto Scaling group spans multiple AZs for redundancy.

Step 7: Security Measures
Configure Security
    Security Groups: Set up security groups in EC2 to control traffic.
    NACLs: Configure NACLs for additional network-level security.
    AWS Systems Manager: Use it for patch management.

Step  8: Monitoring and Alerts
CloudWatch Setup
    Navigate to CloudWatch: Set up CloudWatch for monitoring.
    Create Alarms: Define alarms based on metrics like CPU utilization, network throughput, etc.

Step 9: Logging and Tagging
Implement Logging
    CloudWatch Logs: Set up logging for your application in CloudWatch.
    Resource Tagging: Tag your AWS resources for better management.

Step 10: Testing and Verification
Final Testing
    Upload JSON: Upload JSON files to your S3 bucket.
    Test Autoscaling: Generate traffic to test the autoscaling setup.
    Monitor: Check CloudWatch for logs and alarms.

Q23. You are tasked with deploying a Python Flask web application that offers a service for image processing and storage. The application needs to be scalable, secure, and well-monitored. You must set up the infrastructure to achieve this. Tasks:

1. Flask Application Deployment: Deploy a basic Python Flask application on AWS. Ensure it's accessible over HTTP. You can use a sample Flask app for this purpose.
2. Autoscaling Configuration: Set up autoscaling for the Flask application. Define a scaling policy that adds or removes instances based on CPU utilization. Test the autoscaling by simulating traffic load on the application.
3. High Availability Setup: Deploy the Flask application across two AWS Availability Zones (AZs) to ensure high availability. Configure a load balancer to distribute incoming traffic evenly between instances in different AZs.
4. Security Measures: Implement security best practices. Ensure that communication with the Flask application is secure over HTTPS. Implement access controls to restrict unauthorized access. Create a security group to control incoming and outgoing traffic to the instances.
5. Monitoring and Alerts: Configure CloudWatch to monitor the Flask application. Set up alarms for key performance metrics such as CPU utilization, memory usage, and request latency. Define actions to be taken when alarms are triggered.
6. Logging and Storage: Set up logging for the Flask application using CloudWatch Logs. Ensure that logs are stored securely and can be easily retrieved for troubleshooting. Implement Amazon S3 for storing processed images from the application.
7. Testing and Cleanup: Test the entire setup by simulating different traffic loads, monitoring the application's performance, and verifying that scaling and security measures work as expected. After successful testing, clean up all AWS resources to avoid unnecessary charges.

Step 1: Flask Application Deployment
Create a Basic Flask Application
    Develop a Flask App: If you don't have one, create a basic Flask application. You can find numerous simple Flask app examples online.
    Containerize the Flask App: Use Docker to containerize your Flask app. Create a Dockerfile in your application directory.

Deploy on AWS
    Set Up AWS Account: If you don’t have an AWS account, create one.
    EC2 Instance: Launch an EC2 instance where your Flask app will run.
        Choose an AMI (Amazon Machine Image), like Amazon Linux 2.
        Select instance type (e.g., t2.micro for free tier).
        Configure instance details (keep defaults for now).
        Add storage if necessary (default is usually sufficient).
        Add tags, like Name: FlaskApp.
        Configure Security Group to allow HTTP (port 80) and SSH (port 22) access.
        Review and launch the instance, and then create a new key pair. Download this key pair, as it's needed for SSH access.
    
    Connect to EC2 Instance via SSH:
        bash
        ssh -i /path/to/your-key.pem ec2-user@your-instance-public-dns
    
    Install Docker on EC2:
        Update the package index: sudo yum update -y
        Install Docker: sudo yum install docker -y
        Start Docker: sudo service docker start
        
    Deploy Flask App on EC2:
        Transfer your application files to EC2 using SCP or Git.
        Build your Docker container: docker build -t flask-app .
        Run your Docker container: docker run -p 80:5000 flask-app

Step  2: Autoscaling Configuration
Set Up Auto Scaling Group:
    Navigate to EC2 Dashboard > Auto Scaling > Create Auto Scaling group.
    Follow the wizard to create a Launch Configuration or Template.
    Define the scaling policy based on CPU Utilization.

Test Autoscaling:
    You can use a tool like Apache JMeter or AWS CloudFormation to simulate traffic load.
    Monitor the scaling activities in the Auto Scaling Groups dashboard.

Step 3: High Availability Setup
Create Multiple EC2 Instances in different Availability Zones.

Set Up Elastic Load Balancer (ELB):
    Navigate to EC2 Dashboard > Load Balancers > Create Load Balancer.
    Choose Application Load Balancer for HTTP/HTTPS traffic.
    Configure load balancer and listeners, and assign it to the created Auto Scaling Group.

Step  4: Security Measures
Implement HTTPS:
    Obtain an SSL/TLS certificate via AWS Certificate Manager.
    Add a listener to your Load Balancer for HTTPS (port 443) and associate the certificate.

Set Up Security Groups and Network Access Control Lists (NACLs) to control inbound and outbound traffic.

Implement IAM Roles for EC2 to grant necessary permissions.

Step  5: Monitoring and Alerts
Configure CloudWatch:
    Navigate to the CloudWatch dashboard.
    Set up monitoring for metrics like CPU Utilization, Memory Usage, etc.
    Create alarms and define actions (like notifications or auto-scaling triggers).

Step  6: Logging and Storage
Integrate CloudWatch Logs for logging.

Set Up Amazon S3 for storing images:
    Create an S3 bucket.
    Modify your Flask application to save processed images to this bucket.

Step 7: Testing and Cleanup
Test Your Setup: Simulate different traffic scenarios and monitor performance.

Verify Security: Ensure that the security measures are working (e.g., accessing via HTTPS, security group rules).

Cleanup: After testing, terminate resources you no longer need to avoid charges.

Q24. You are a DevOps engineer responsible for managing the deployment pipeline and infrastructure for a web application. The application consists of a frontend and backend, both hosted on Amazon EC2 instances. Your goal is to establish a robust CI/CD pipeline and automate the deployment using AWS CloudFormation. Additionally, you need to set up monitoring and logging to ensure the health and performance of the application. Tasks:

1. CI/CD Pipeline Setup:
- Implement a CI/CD pipeline using a CI/CD service of your choice (e.g., AWS CodePipeline, Jenkins). Connect it to your version control system (e.g., GitHub).
- Configure the pipeline to trigger on code commits to the repository.
- Set up build and deploy stages for both the frontend and backend components. 
2. EC2 Instances and Autoscaling:
- Launch Amazon EC2 instances for the frontend and backend using AWS CloudFormation. Define the necessary parameters, resources, and outputs in the CloudFormation template.
- Implement an Auto Scaling group for the backend to ensure redundancy and scalability. Configure appropriate launch configurations and policies. 
3. Code Deployment:
- Automate the deployment of frontend and backend code using your CI/CD pipeline. Ensure that new code versions are deployed to EC2 instances without manual intervention.
4. Monitoring and Logging:
- Configure CloudWatch Alarms to monitor key metrics (e.g., CPU utilization, network traffic) for your EC2 instances. Define meaningful thresholds and actions for each alarm.
- Set up CloudWatch Logs for both frontend and backend components. Ensure that logs are centralized and easily accessible for troubleshooting. 
5. Application Health Checks:
- Implement custom health checks for your application. Define an endpoint that returns the health status of the frontend and backend. Configure the load balancer to use these health checks.
6. Rollback Mechanism:
- Create a rollback mechanism in your CI/CD pipeline. Define conditions that trigger a rollback (e.g., failed health checks, high error rates). Ensure that the previous version is automatically restored.
7. Testing and Validation:
- Test the pipeline by making code changes and observing the deployment process. Verify that the application is running as expected on the EC2 instances.
8. Clean-Up and Documentation:
- Document the entire setup, including CI/CD pipeline configuration, CloudFormation template, and monitoring setup. - After successful testing, clean up any unused resources to avoid unnecessary charges.

Step 1: Create an IAM Roles.
Go to IAM in the AWS Management Console.
Create role
    Trusted entity type: AWS service
    Use case: EC2

    Next -> Add permissions
    Select "AmazonEC2RoleforAWSCodeDeploy"

    Next -> Role details
    Role name: ec2pipelinerole

    keep rest settings default
    create

Create role
    Trusted entity type: AWS service
    Use case: CodeDeploy

    Next -> Add permissions
    Select "AWSCodeDeployRole"

    Next -> Role details
    Role name: codedeployrole

    keep rest settings default
    create

Step 2: Create instance
Go to EC2 in AWS Management Role
Navigate to Instance in Instances section on left sidebar
Launch instance
    Name and tags
    Name: CICDInstance
    Application and OS Images (Amazon Machine Image): Amazon Linux 2
    Instance type: t2.micro
    Key pair (login)
    Key Pair: demo-server (Use the key pair generated)
    Network settings
    Firewall (security groups): Select existing security group (default)
    IAM Role: ec2pipelinerole
    User data:
        #!/bin/bash
        sudo yum -y update
        sudo yum -y install ruby
        sudo yum -y install wget
        cd /home/ec2-user
        wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install
        sudo chmod +x ./install
        sudo ./install auto
        sudo yum install -y python-pip
        sudo pip install awscli
    keep rest all settings default

Step 3: Create an Application
Go to the CodePipeline in the AWS Management Console.
Navigate to Applications in Deploy section on left sidebar
Create application
    Application name: ec2application
    Compute platform: EC2

Create deployment group
    Enter a deployment group name: ec2appdp
    Service role: codedeployrole
    Deployment type: In-place

    Environment configuration
    Select Amanzon EC2 instances
    tag -> ec2application

    keep rest settings default

Navigate to Pipeline on left sidebar
Create pipeline
    Pipeline name: ec2pipeline
    Service role: New service role

    Next -> Add source stage
    Source provider: GitHub (Version 2)
    Connect to GitHub
        Connection name: ec2connection

        Next -> Connect to GitHub
        GitHub Apps: Select the user
        Connect
    
    Repository name: Select the repo
    Branch name: main

    Next -> Add build stage
    Skip build stage

    Next -> Add deploy stage
    Deploy provider: AWS CodeDeploy
    Region: use the region where the application is created
    Application name: ec2application
    Deployment group: ec2appdp

    Next -> Create pipeline

Step 3: Go back to EC2
Navigate to the instance you created
Select the instance and go in Security and click on Security Groups
Edit inbound rules
Add rule
    Type: HTTP
    Source: Anywhere IPv4

    Type: SSH
    Source: Anywhere IPv4
    Save rules

Step 4: Upload files to GitHub

YAML File: ec2app.yml
version: 0.0
os: linux
files:
  - source: /index.html
    destination: /var/www/html/
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies
      timeout: 300
      runas: root
    - location: scripts/start_server
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server
      timeout: 300
      runas: root

scripts/install_dependencies
#!/bin/bash
yum install -y httpd

scripts/start_server
#!/bin/bash
service httpd start

scripts/stop_server
#!/bin/bash
isExistApp = `pgrep httpd`
if [[ -n  $isExistApp ]]; then
    service httpd stop        
fi

index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EC2 Application</title>
</head>

<style>
    body{
        font-family: Verdana, Geneva, Tahoma, sans-serif;
    }
</style>
<body>
    <h1 style="text-align: center;"><b>EC2 Application</b></h1>
    <h3>This application was deployed using AWS CodeDeploy.</h3>
    <p>Hello World </p>
</body>
</html>

Step 4. Code Deployment Automation
    In the CodePipeline setup, add deploy stages.
    Utilize AWS CodeDeploy to automate deployments to EC2 instances.
    Ensure CodeDeploy Agents are installed on your EC2 instances.

Step 5: Monitoring and Logging with CloudWatch
    Configure CloudWatch Alarms
        Navigate to CloudWatch.
        Create alarms for CPU utilization, network traffic, etc.
        Set thresholds and actions (like notifications).
    Set up CloudWatch Logs
        Ensure application logging is configured to send logs to CloudWatch.
        Create log groups and streams in CloudWatch.

Step 6: Application Health Checks
    Implement health check endpoints in your application.
    Configure the load balancer to use these endpoints for health checks.

Step 7: Rollback Mechanism
    In CodePipeline, configure rollback actions based on certain conditions.
    Utilize AWS CodeDeploy’s rollback features on deployment failure.
    
Step 8: Testing and Validation
    Make a minor code change in your repository.
    Push the change and observe the pipeline executing build and deploy.
    Verify the application is running as expected on EC2 instances.

Step 9: Clean-Up and Documentation
    Document each step, including pipeline configuration, CloudFormation template details, and monitoring setup.
    After testing, remove any unnecessary resources to prevent extra charges.

Q25. You are responsible for setting up a CI/CD pipeline for a web application on AWS. The application runs on EC2 instances and is defined in CloudFormation templates (CFT). You must also ensure proper monitoring and logging of the application's performance and issues. Tasks:
1. CI/CD Pipeline Setup: Create a CI/CD pipeline using AWS CodePipeline and AWS CodeBuild. The pipeline should pull the application code from a source code repository (e.g., GitHub), build the code, and deploy it to EC2 instances.
2. EC2 Instances Deployment: Launch EC2 instances using CloudFormation templates (CFT). Define the CFT for EC2 instances, including necessary security groups, IAM roles, and user data to install and configure the application.
3. Pipeline Integration: Integrate the CI/CD pipeline with the CloudFormation stack that deploys EC2 instances. Ensure that the pipeline automatically deploys the application to new instances when changes are made to the source code.
4. Monitoring Configuration: Configure AWS CloudWatch to monitor EC2 instances. Set up custom metrics to monitor application-specific parameters, such as response times, error rates, and resource utilization.
5. CloudWatch Alarms: Set up CloudWatch alarms to alert you when certain conditions are met, such as high CPU utilization or excessive error rates. Define actions to be taken when alarms are triggered.
6. Logging and Troubleshooting: Configure EC2 instances to send application logs to CloudWatch Logs. Ensure that logs are stored securely and can be accessed for troubleshooting and analysis.
7. Pipeline Testing: Test the CI/CD pipeline by making changes to the application code and verifying that the pipeline automatically deploys the changes to EC2 instances. Monitor the process to ensure it works as expected.

Step 1: CI/CD Pipeline Setup with AWS CodePipeline and AWS CodeBuild
AWS CodePipeline Setup
    Log into AWS Console and navigate to the CodePipeline service.
    Create Pipeline:
        Pipeline name: Choose a meaningful name.
        Service role: Select 'New service role' if you don't have one.
        Artifact store: Use the default location or specify a custom S3 bucket.
        
        Next -> Add source stage:
        Source Provider: Select 'GitHub' or your preferred source repository.
        Connect to GitHub: Follow prompts to connect your GitHub account.
        Choose Repository and Branch: Select your application’s repository and branch.

        Next -> Add build stage
        Build Stage with AWS CodeBuild:
            Add Build Stage: In the pipeline, add a new stage for building.
            Create a build project:
                Project Name: Enter a name.
                Environment: Choose an environment image, typically a managed image.
                Service Role: Create or select an existing role.
                Buildspec: Define your build commands either in a buildspec.yml file in your repository or inline.
        
        Deploy Stage:
            Deployment Provider: Choose 'AWS CloudFormation'.
            Action Mode: Select 'Create or replace a change set'.
            Stack Name: Name of your CloudFormation stack.
            Change set name: Name your change set.
            Template: Provide the path to your CloudFormation template in your repository.

Step 2: EC2 Instances Deployment using CloudFormation
Create CloudFormation Template (CFT)
    Define EC2 Instances: Specify instance type, AMI ID, and key pair.
    Define Security Groups: Set inbound and outbound rules for your instances.
    Define IAM Roles: Create roles for EC2 with necessary permissions.
    User Data: Script to install and configure the application on EC2 instances.
    Save the CFT: Save the template in your source repository.

Step 3:Pipeline Integration with CloudFormation
Ensure your pipeline’s deploy stage is properly set to deploy to EC2 instances as defined in your CFT.

Step 4: Monitoring Configuration with AWS CloudWatch
Setup CloudWatch Monitoring
    Navigate to CloudWatch: In the AWS Console.
    Create Custom Metrics: For application-specific parameters.
    Configure EC2 for CloudWatch: Ensure EC2 instances are sending metrics to CloudWatch.

Step 5: CloudWatch Alarms Setup
Create Alarms: In CloudWatch, create alarms for conditions like high CPU usage.
Define Actions: Set actions like notifications for when alarms are triggered.

Step 6: Logging and Troubleshooting
Configure CloudWatch Logs
    Set Up Log Groups: In CloudWatch, for your application.
    Configure EC2: Ensure instances are sending logs to CloudWatch Logs.
    Test Logging: Verify that logs are being sent and stored correctly.

Step 7: Pipeline Testing
Make Changes in Source Repository: Update the application code.
Commit and Push Changes: Push changes to your repository.
Monitor Pipeline: Check the pipeline executes and deploys changes.
Verify Deployment: On EC2 instances.
Monitor CloudWatch: Check for logs and metrics.

Q26. You are a cloud engineer tasked with setting up a Virtual Private Cloud (VPC) to host a web application. The application will run on an Amazon EC2 instance. Your goal is to ensure that the VPC is properly configured and that the EC2 instance is launched and accessible. Tasks:
1. VPC Creation: Create a new VPC with the following specifications:
VPC CIDR Block: 10.0.0.0/16
Public Subnet: 10.0.0.0/24 (us-east-1a)
Private Subnet: 10.0.1.0/24 (us-east-1b)
Internet Gateway: Attach an Internet Gateway to the VPC for internet access.
2. Route Table Configuration: Create two route tables - one for the public subnet and one for the private subnet. Ensure that the public subnet's route table has a route to the Internet Gateway.
3. Security Group Setup: Create a security group that allows inbound traffic on port 80 (HTTP) for the web application. Attach this security group to both the public and private subnets.
4. Key Pair Generation: Create an EC2 key pair for SSH access to the instances. Store the private key securely. 5. EC2 Instance Launch: Launch an Amazon EC2 instance in the public subnet with the following specifications:
Amazon Machine Image (AMI): Amazon Linux 2
Instance Type: t2.micro
Security Group: Use the security group created in step 3.
Key Pair: Use the key pair generated in step 4.
IAM Role: Attach an IAM role that allows basic EC2 permissions.
6. Public IP Assignment: Ensure that the EC2 instance has a public IP address.
7. Web Application Deployment: SSH into the EC2 instance and deploy a sample web application. You can use a basic HTML file or any sample application of your choice.
8. Access Verification: Access the web application via the public IP address of the EC2 instance using a web browser to verify that the deployment is successful.

Go to VPC in the AWS Management Console. 
Create VPC (Region: us)
    Select VPC only
    Name tag - optional: vcc-vpc
    VPC CIDR Block: 10.0.0.0/16
    and keep other settings default
    Create

Navigate to Subnets on left sidebar
Subnets
Create subnets
    VPC
    VPC ID: vcc-vpc
    Subnet settings
    Subnet 1 of 2
    Subnet name: Public-1a
    Availability Zone: us-east-1a
    IPv4 subnet CIDR block: 10.0.0.0/24 
    Subnet 2 of 2
    Subnet name: Private-1a
    Availability Zone: us-east-1b
    IPv4 subnet CIDR block: 10.0.1.0/24

Navigate to Internet gateways on left sidebar
Create a internet gateway
    Name tag: vcc-internet-gateway
    After creating a internet gateway -> Actions
    Actions -> Attach to VPC
    VPC -> Available VPCs: vcc-vpc

Navigate to Route table on left sidebar
Create route table
Route table settings
    Name - optional: vcc-route-table
    VPC: vcc-vpc
    after creating the route table, select the route table and go to Subnet associations
    Edit subnet association
        Select public subnet: Public-1a
        Save association
    after saving the subnet association, select the route table and go to Routes
    Edit Routes
        Add route
        Search: 0.0.0/0
        Select: Internet Gateway -> prac-internet-gateway
        Save changes
   
Navigate to Security groups in the Security section on left sidebar
Create security group
    Basic details
    Security group name: VCCSecurityGroup
    Description: VCC Exam
    VPC: vcc-vpc

    Inbound rules
    Add rule
        Type: HTTP
        Soure: My IP

Go to EC2 in the AWS Management Console. 
Navigate to Key Pairs in Network & Security section on left sidebar
Create key pair
    Name: prac-key-pair
    Key pair type: RSA
    Private key file format: .pem
    Create

Navigate to Instance in Instances section on left sidebar
Launch instance
    Name and tags
    Name: VCCInstance
    Application and OS Images (Amazon Machine Image): Amazon Linux 2
    Instance type: t2.micro
    Key pair (login)
    Key Pair: prac-key-pair (Use the key pair generated)
    Network settings
    Firewall (security groups): Select existing security group
    VPC - required: vcc-vpc
    Subnet: Public-1a
    Auto-assign public IP: Enable
    Common security groups: VCCSecurityGroup (Use the security group created)
    IAM Role: vcc-vpc-role
    keep rest all settings default

Go to AWS System Manager in the AWS Management Console. 
Navigate to Session Manager in Actions section on left sidebar
Start session
Select the Instance
    sudo yum install -y httpd
    sudo yum install -y wget
    cd /var/www/html
    vi index.html & write "Hello World!" > index.html
    sudo service httpd start

Q27. You are a cloud engineer tasked with setting up a Virtual Private Cloud (VPC) and an Amazon EC2 instance in AWS. Your goal is to create a secure network environment and verify the correct deployment of the EC2 instance. Tasks:

1. VPC Creation: Create a new Virtual Private Cloud (VPC) using the AWS Management Console. Configure the VPC with a unique IPv4 CIDR block. Ensure that the VPC is located in a specific AWS region.
2. Subnet Setup: Create two subnets within the VPC. One subnet should be public and the other private. Associate an Availability Zone with each subnet. Define appropriate IPv4 CIDR blocks for each subnet.
3. Internet Gateway: Create an internet gateway and attach it to the VPC. Modify the route table of the public subnet to allow traffic to and from the internet via the internet gateway.
4. Security Groups: Configure security groups for the EC2 instance. Define rules for incoming and outgoing traffic to ensure that the EC2 instance is accessible over SSH (port 22) and HTTP (port 80).
5. Key Pair: Create an EC2 key pair to use for securely accessing the instance over SSH.
6. EC2 Instance Launch: Launch an Amazon EC2 instance within the public subnet. Choose a suitable Amazon Machine Image (AMI) and instance type. Use the previously created security group and key pair. Assign an Elastic IP address to the instance.
7. Connect to EC2: Connect to the EC2 instance using SSH to verify that it is running and correctly deployed. Ensure that it is accessible via its public IP address.

Go to VPC in the AWS Management Console. 
Create VPC (Region: us)
    Select VPC only
    Name tag - optional: vcc-vpc
    VPC CIDR Block: 10.0.0.0/16
    and keep other settings default
    Create

Navigate to Subnets on left sidebar
Subnets
Create subnets
    VPC
    VPC ID: vcc-vpc
    Subnet settings
    Subnet 1 of 2
    Subnet name: Public-1a
    Availability Zone: us-east-1a
    IPv4 subnet CIDR block: 10.0.0.0/24 
    Subnet 2 of 2
    Subnet name: Private-1a
    Availability Zone: us-east-1b
    IPv4 subnet CIDR block: 10.0.1.0/24

Navigate to Internet gateways on left sidebar
Create a internet gateway
    Name tag: vcc-internet-gateway
    After creating a internet gateway -> Actions
    Actions -> Attach to VPC
    VPC -> Available VPCs: vcc-vpc

Navigate to Route table on left sidebar
Create route table
Route table settings
    Name - optional: vcc-route-table
    VPC: vcc-vpc
    after creating the route table, select the route table and go to Subnet associations
    Edit subnet association
        Select public subnet: Public-1a
        Save association
    after saving the subnet association, select the route table and go to Routes
    Edit Routes
        Add route
        Search: 0.0.0/0
        Select: Internet Gateway -> prac-internet-gateway
        Save changes
   
Navigate to Security groups in the Security section on left sidebar
Create security group
    Basic details
    Security group name: VCCSecurityGroup
    Description: VCC Exam
    VPC: vcc-vpc

    Inbound rules
    Add rule
        Type: SSH
        Source: My IP
        Type: HTTP
        Soure: My IP

Go to EC2 in the AWS Management Console. 
Navigate to Key Pairs in Network & Security section on left sidebar
Create key pair
    Name: prac-key-pair
    Key pair type: RSA
    Private key file format: .pem
    Create

Navigate to Instance in Instances section on left sidebar
Launch instance
    Name and tags
    Name: VCCInstance
    Application and OS Images (Amazon Machine Image): Amazon Linux 2
    Instance type: t2.micro
    Key pair (login)
    Key Pair: prac-key-pair (Use the key pair generated)
    Network settings
    Firewall (security groups): Select existing security group
    VPC - required: vcc-vpc
    Subnet: Public-1a
    Auto-assign public IP: Enable
    Common security groups: VCCSecurityGroup (Use the security group created)
    IAM Role: vcc-vpc-role
    keep rest all settings default

Navigate to Elastic IPs in Network & Security section on left sidebar
Allocate Elastic IP address
Select the Allocated Elastic IP address -> Actions -> Associate Elastic IP address
    Resource type: Instance
    Instance: VCCInstance
    Private IP address: Private-1a
    Associate

Q28. Create and Configure an EC2 Instance:
1. Launch a new EC2 instance using the AWS CLI, specifying instance type, key pair, and security group. 
2. Attach an Elastic IP address to the EC2 instance.
3. SSH into the EC2 instance using the key pair.

Step 1: Go to E2 in AWS Management Console.
Launch Instance

aws ec2 run-instances
 --image-id ami-0230bd60aa48260c6
 --instance-type t2.micro
 --key-name demo-server
 --security-group-ids sg-0d7c062613b96b6fc
 --subnet-id subnet-098a3299f03c75e7b

"Groups": [],
"Instances": [
    {
        "AmiLaunchIndex": 0,
        "ImageId": "ami-02a2af70a66af6dfb",
        "InstanceId": "i-057fbdd0flec4bcad",
        "InstanceType": "t2.micro"
        "KeyName": "demo-server"
        "LaunchTime": "2023-11-16T18:31:12+00:00",
        "Monitoring": {
            "State": "disabled"
        },
        "Placement": {
            "AvailabilityZone": "ap-south-1b",
            "GroupName": ""
            "Tenancy": "default"
        },
        "PrivateDnsName": "ip-172-31-0-187-ap-south-1. compute.internal",
        "PrivateIpAddress": "172.31.0.187"
        "ProductCodes": L],
        "PublicDnsName": "",
        "State": {
            "Name": "pending"
        },
        "StateTransitionReason": ""
        "SubnetId": "subnet-0b3cd6235fe0b3310"
        "VpcId": "vpc-043e6209e4e2f9642"
        "Architecture": "×86_64"
        "BlockDeviceMappings": [],
        "ClientToken": "167afe45-a454-46f1-a795-e10a71ba85fd",
        "EbsOptimized": false,
        "EnaSupport": true,
        "Hypervisor": "xen"
        "NetworkInterfaces": [

    aws ec2 allocate-address --domain vpc

    aws ec2 associate-address \
    --instance-id i-057fbdd0f1ec4bcad \  # Specify your EC2 instance ID
    --allocation-id eipalloc-0fb91febd89c7c36e  # Replace with the AllocationId obtained in the previous step

    ssh ec2-user@13.127.192.239 #you will get ip address in above command.
Q29. Create an S3 Bucket, Upload, and Download Files:
1. Create a new S3 bucket with a unique name.
2. Upload a local file to the S3 bucket.
3. Download the file from the S3 bucket to your local machine.

Step 1: Create an S3 Bucket.
Go to S3 in the AWS Management Console.
    Create bucket
    Bucket name: vcc.bucket
    AWS Region: Asia Pacific(Mumbai) ap-south-1 (select the AWS Region you want to create your bucket in)
    and keep all other settings as default for now.

Step 2: Upload a local file in S3 Bucket
Go into Buckets from left sidebar
    Click on the bucket which you just created: vcc.bucket
    Click on Upload
    Files and folders -> Add files
    Upload

Step 3: Download the file from S3 Bucket.
Go into Buckets from left sidebar.
    Click on the bucket which you just created: vcc.bucket.
    Click on the file which you just uploaded: vcc.bucket.
    And above that file in the menu bar you can see the option to download.

Q30. Configure SNS and SQS for Messaging:
1. Create an SNS topic using the AWS CLI.
2. Create an SQS queue and subscribe it to the SNS topic.
3. Send a test message to the SNS topic and verify that it's delivered to the SQS queue.

Create a SNS topic  
    aws sns create-topic --name MySNSTopic

Create SQS quese
    aws sqs create-queue --queue-name MySQSQueue

Go to console and copy Queue ARN Or use  this command, 
arn=$(aws sqs get-queue-attributes --queue-url YourSQSQueueURL --attribute-names QueueArn --output text --query 'Attributes.QueueArn')

aws sns subscribe --topic-arn <Your SNS Topic ARN> --protocol sqs --notification-endpoint $arn

Send message
aws sns publish --topic-arn <TOPIC ARN> --message ‘Hello, this is a test message

Check whether message received or not
aws sqs receive-message --queue-url YourSQSQueueURL

Q31. You are tasked with deploying a static website for a small business using Amazon S3. The website will contain basic HTML, CSS, and JavaScript files. Your goal is to create an S3 bucket, configure it for static website hosting, upload the website files, and verify its accessibility.

Step 1: Create an S3 Bucket.
Go to S3 in the AWS Management Console.
    Create bucket
    Bucket name: vcc.bucket
    AWS Region: Asia Pacific(Mumbai) ap-south-1 (select the AWS Region you want to create your bucket in)
    keep all other settings as default for now.
    and disable Block all public access
    Create bucket

After creation of the bucket select the bucket and open the properties of the bucket.
Properties
    Static Website Hosting: Enable
    Hosting type: Host a static website
    Index document: index.html

then navigate to Permissions
Permissions
Object Ownership: ACLs enabled

Now go in the objects
    Select all objects
    Actions -> Make public using ACL