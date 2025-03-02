# serverless-microservices
RESTful Microservices with AWS Lambda, API Gateway and DynamoDB
Let's start with the High Level Reference architecture for RESTful microservices.

![Screenshot 2025-03-01 at 8 46 23 PM](https://github.com/user-attachments/assets/ad3f0f8a-474a-49d0-8a5d-45dbc518d6f3)


1- Clients send request our microservices by making HTTP API calls. Ideally, our clients should have a tightly bounded service contract to our API in order to achieve consistent expectations of microservice responsibility.

2- Amazon API Gateway hosts RESTful HTTP requests and responses to customers. In this scenario, API Gateway provides built-in authorization, throttling, security, fault tolerance, request/response mapping, and performance optimizations.

3- AWS Lambda contains the business logic to process incoming API calls and leverage DynamoDB as a persistent storage.

4- Amazon DynamoDB persistently stores microservices data and scales based on demand. Since microservices are often designed to do one thing well, a schemaless NoSQL data store is regularly incorporated.

So When we invoke our API Gateway API, API Gateway routes the request to your Lambda function. The Lambda function interacts with DynamoDB, and returns a response to API Gateway. API Gateway then returns a response to us.

For this exercise, you create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function. That is, when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

Create, update, and delete an item.
Read an item.
Scan an item.
Other operations (echo, ping), not related to DynamoDB, that you can use for testing.

The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:

![1740884314110](https://github.com/user-attachments/assets/e5fe1962-689e-4d32-9462-8d92f05fe718)

The following is a sample request payload for a DynamoDB read item operation:

![1740884314130](https://github.com/user-attachments/assets/1e801c8b-62e4-4cb3-83ae-62fdbce0352d)

We will then use Postman API platform to run performance test on our Lambda and recording the response time based on its memory configuration.

Setup

Create Lambda IAM Role

Create the execution role that gives your function permission to access AWS resources.

To create an execution role

Open the policies page in the IAM console.
Choose Create policy.
Create a policy with the following properties.Policy name – lambda-apigateway-dynamodb-policy.Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to DynamoDB and upload logs.




Click Create Policy
Select Roles and click "Create Roles"
Select "Trusted entity type" as AWS Service
Select your service as "Lambda"
Select "lambda-apigateway-dynamodb-policy" as your permission policy
Enter role name as "lambda-apigateway-role"
Select, "Create role"

Create Lambda Function

To create the function

Click "Create function" in AWS Lambda Console


Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.12 as Runtime. Under Permissions, select "Use an existing role", and select lambda-apigateway-role that we created, from the drop down
Click "Create function"


Replace the boilerplate coding with the following code snippet and click "Save"

Example Python Code



Test Lambda Function

Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.

Click the arrow on "Test" and click "Configure test events"






Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Create" to save


Click "Test", and it will execute the test event. You should see the output in the console.




We're all set to create DynamoDB table and an API using our lambda as backend!

Create DynamoDB Table

Create the DynamoDB table that the Lambda function uses.

To create a DynamoDB table

Open the DynamoDB console.
Choose Create table.
Create a table with the following settings.Table name – lambda-apigatewayPrimary key – id (string)
Choose Create.




Create API

To create the API

Go to API Gateway console
Click Create API




Scroll down and select "Build" for REST API




Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"




Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next

Click "Actions", then click "Create Resource"


Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create Resource"


Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Create Method".




Select "POST" from drop down , then click checkmark






The integration will come up automatically with "Lambda Function" option selected. Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name, your function name will show up.Select and click "Save". A popup window will come up to add resource policy to the lambda to be invoked by this API. Click "Ok"

Our API-Lambda integration is done!

Deploy the API

In this step, you deploy the API that you created to a stage called prod.

Click "Actions", select "Deploy API"


Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"




We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen




Running our solution

The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:






To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity.
To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200




To run this from terminal using Curl, run the below

$ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Bob\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Explore table items" tab, and the newly inserted item should be displayed.


To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table



We have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!

Performance test using Postman

Install native desktop version postman tool and sign in to an account.
Create new collection and post method.
Run collection in performance test mode.




Below is the report for the average response time and error rate where the configuration for Lambda function was 128 MB Memory and 10 sec time out.




Then I updated the Lambda configuration by increasing 1024 MB Memory and 10 sec time out and now if I run the performance test you can see the average response time is 283 ms (decreased) as per below performance report.




Cleanup

Let's clean up the resources we have created for this lab.

Cleaning up DynamoDB

To delete the table, from DynamoDB console, select the table "lambda-apigateway", and click "Delete"




To delete the Lambda, from the Lambda console, select lambda "LambdaFunctionOverHttps", click "Actions", then click Delete


To delete the API we created, in API gateway console, under APIs, select "DynamoDBOperations" API, click "Actions", then "Delete"




