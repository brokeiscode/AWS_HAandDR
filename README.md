## High Availability and Disaster Recovery with AWS Serverless Architecture

## Lab Overview

This lab demonstrates how to build a highly available and disaster-resilient application using AWS serverless services. The solution includes:
- **DynamoDB Global Tables:** for cross-region replication.
- **AWS Lambda:** (Python) for serverless compute.
- **API Gateway:** to expose REST APIs.
- **Amazon Route 53:** for DNS-based failover.
- A **S3 Static Website:** (HTML + CSS + JavaScript) to interact with the APIs.

## Setup

**1: Set Up DynamoDB Global Tables**
- From **AWS Management Console**, navigate to **DynamoDB**.
- Create a new table in the primary region of your choice.
- Next, navigate to the **Global Tables** tab and click **Create replica** in a secondary region of your choice.
- Wait for the replica to be active.

##

**2: Create a Lambda Function Execution role**
- Navigate to **IAM** from the **AWS Management Console**.
- Click **Create Role**, select **Trusted entity type** as AWS Service and **Use case** as Lambda.
- Attach the following policies:
    - `AmazonDynamoDBFullAccess` (for development phase, for production work - use IAM Access Analyzer to help identify the required permissions for the IAM execution role policy).
    - `AWSLambdaBasicExecutionRole` (for CloudWatch logging).
- Next, Input a **Role name** and click **Create Role**
- Note the role ARN.

##

**3: Create two Lambda Functions in each of the Regions**
- Create two Lambda functions in Python: one for reading data and one for writing data (samples available this repository).
- The **Read Data Function** is `read_function.py` and the **Write Data Function** is `write_function.py`.
- In the primary region, navigate to **AWS Lambda** from the **AWS Management Console**.
- Click **Create function**:
    - Select **Runtime** as Python 3.9
    - Click **Change default execution role** and select **Use an existing role**
    - Search the name of the execution role you created above and select it.
    - Update the **Code** with the `read_function.py` file in this repository (copy and paste).
    - Edit code and input correct names.
    - **Deploy**
- Repeat the entire above process for the second lambda function, using the `write_function.py` code.
- Next, repeat and deploy the two functions in the Secondary region.

##

**4: Set Up API Gateway in Both Regions**
- Navigate to the **API Gateway** console.
- Create a new REST API:
    - Create two resources: `/read` and `/write`.
    - Enable CORS (Cross Origin Resource Sharing).
    - Link `/read` to **Read Data Function** with a `GET` method.
    - Link `/write` to **Write Data Function** with a `POST` method.
    - Use proxy integrations.
- Deploy the API to a new stage (e.g., `prod`).
- Note the API Gateway endpoint URL.
- Repeat the same setup in the secondary region.

##

**5: Create ACM certificates and custom domain for the APIs**
- Navigate to the **Certificate Manager** console.
- Request a certificate with your domain name "api.<YOUR-DOMAIN-NAME>" and use DNS validation.
- Configure Custom domain names in API Gateway, select the appropriate certificate.
- Create an API mapping selecting your API and stage.
- Repeat the same steps for the second Region.

##

**6: Set Up Route 53 DNS Record and Health Check**
- Go to the **Route 53** console.
- Create health checks with the /read endpoint
- Create a new Route 53 record set:
    - **Routing policy** as `Failover`
    - **Primary endpoint** with primary region API Gatewat URL
    - **Secondary endpoint** with secondary region API Gatewat URL
- Configure health checks to match your failover requirements

##

**7: Create the S3 Static Website**
- A static website to interact with the API.
- Using the `index.html` in the repository.
- update the html file with your DNS name.
- create an S3 bucket, upload the updated `index.html` file and set it up for static hosting.

##

**8: Test the Disaster Recovery**
- **Simulate a Failure**: Delete the API Gateway in the primary region.
- Route 53 health checks will detect the failure and route traffic to the secondary region.

##
## System is High Available and Disaster Recovery Tested.
