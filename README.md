# Use lambda function to add csv into DynamoDB from S3 bucket

Overview: Goal of this project is to add a CSV file with list of organizations in the S3 bucket. When the file is uploaded a lambda function will be triggered which will read the csv file and upload the list in Dynamodb table. Below is the list of organizations we are trying to upload:

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled.png)

Step 1: Create a DynamoDB table

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%201.png)

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%202.png)

Step 2: Create a S3 bucket to upload the csv file:

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%203.png)

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%204.png)

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%205.png)

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%206.png)

Upload the file into the S3 bucket:

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%207.png)

Step 3: Create a Lambda function:

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%208.png)

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%209.png)

Create a role that can allow the access to the S3 bucket and DynamoDB. Fine-grained permissions, can created for a custom IAM policy that allows only the necessary actions like dynamodb:PutItem.

Make sure the lambda function is created in the same region as the S3 bucket was created.

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2010.png)

This is the python code for a Lambda function that reads data from an S3 bucket triggered by an event and inserts it into a DynamoDB table.

```bash
import json
import boto3

s3_client = boto3.client('s3')
dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table('organization_list')

def lambda_handler(event, context):
    try:
        
        bucket_name = event["Records"][0]["s3"]["bucket"]["name"]
        s3_file_name = event["Records"][0]["s3"]["object"]["key"]
        
        response = s3_client.get_object(Bucket=bucket_name, Key=s3_file_name)
        data = response["Body"].read().decode('utf-8')

        organizations = data.split("\n")
        for org in organizations:
            org = org.split(",")
            table.put_item (
                Item = {
                    "Index": str(org[0]),
                    "Organization Id": str(org[1]),
                    "Name": str(org[2])
            }
        )
    except Exception as err:
        print (err)
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda')
    }
```

Chatgpt prompt to for the boto3 code : "Retrieve the code for a Lambda function that reads data from an S3 bucket triggered by an event and inserts it into a DynamoDB table. The code should include the necessary imports and configurations."

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2011.png)

S3 bucket-name and the S3 file name shall be updated in the event JSON file.

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2012.png)

Error received: "errorMessage": "2023-06-16T14:37:54.136Z feea4a85-4a20-499b-8039-052ca993aca8 Task timed out after 3.01 seconds”. This can be avoided by increasing the timeout value in the lambda function configuration settings.

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2013.png)

If we check the DynamoDB we can see the tables being added from the S3.

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2014.png)

Step 4: Create event notification:

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2015.png)

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2016.png)

![Untitled](Use%20lambda%20function%20to%20add%20csv%20into%20DynamoDB%20from%20%20d405acd81d694757b8939700d365acd8/Untitled%2017.png)