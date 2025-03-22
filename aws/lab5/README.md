# Serverless Application with AWS Lambda, API Gateway, and DynamoDB

This guide provides step-by-step instructions to build a serverless application using AWS Lambda, API Gateway, and DynamoDB. The application supports basic CRUD operations (GET and POST).

## Prerequisites

- AWS account.
- Installed and configured AWS CLI.
- Node.js installed.
- `zip` utility installed for packaging.
- DynamoDB table named `ItemsTable`.

## Steps

### 1. Set Up the DynamoDB Table
1. Navigate to the AWS Management Console.
2. Create a new DynamoDB table:
   - **Table Name**: `ItemsTable`
   - **Partition Key**: `id` (String)

### 2. Create the Lambda Function
1. Create a new directory for your Lambda function:

   ```bash
   mkdir lambda_project && cd lambda_project
   ```

2. Initialize the project and install AWS SDK:

   ```bash
   npm init -y
   npm install aws-sdk
   ```

3. Create the Lambda function file `index.mjs` and paste the following code:

   ```javascript
   import AWS from 'aws-sdk';

   const dynamodb = new AWS.DynamoDB.DocumentClient();

   export const handler = async (event) => {
       console.log('Event:', JSON.stringify(event));

       const httpMethod = event.httpMethod || (event.requestContext ? event.requestContext.http.method : null);

       if (!httpMethod) {
           return {
               statusCode: 400,
               body: JSON.stringify({ message: "Missing httpMethod in event" }),
           };
       }

       try {
           if (httpMethod === 'GET') {
               const params = {
                   TableName: 'ItemsTable',
               };

               const data = await dynamodb.scan(params).promise();
               console.log('Scan result:', data);

               return {
                   statusCode: 200,
                   body: JSON.stringify(data.Items),
               };
           } else if (httpMethod === 'POST') {
               if (!event.body) {
                   return {
                       statusCode: 400,
                       body: JSON.stringify({ message: "Missing body in event" }),
                   };
               }

               const item = JSON.parse(event.body);
               console.log('Item to insert:', item);

               const params = {
                   TableName: 'ItemsTable',
                   Item: item,
               };

               await dynamodb.put(params).promise();

               return {
                   statusCode: 201,
                   body: JSON.stringify({ message: 'Item added successfully' }),
               };
           } else {
               return {
                   statusCode: 400,
                   body: JSON.stringify({ message: 'Unsupported method' }),
               };
           }
       } catch (error) {
           console.error('Error:', error);

           return {
               statusCode: 500,
               body: JSON.stringify({ message: 'Internal Server Error', error: error.message }),
           };
       }
   };
   ```

4. Package the Lambda function:

   ```bash
   zip -r function.zip index.mjs node_modules
   ```

### 3. Deploy the Lambda Function
1. Navigate to the AWS Lambda Console.
2. Create a new function:
   - **Runtime**: Node.js 18.x
3. Upload the `function.zip` file as the function code.
4. Configure the function’s IAM role to include:
   - Permissions for DynamoDB `Scan` and `PutItem`.

### 4. Create the API Gateway
1. Navigate to the API Gateway Console.
2. Create a new HTTP API.
3. Configure routes:
   - **GET /items** -> Integrate with the Lambda function.
   - **POST /items** -> Integrate with the Lambda function.
4. Deploy the API.

### 5. Test the Application

#### Test GET Request
```bash
curl -X GET https://<API_ID>.execute-api.<REGION>.amazonaws.com/prod/items
```

#### Test POST Request
```bash
curl -X POST -H "Content-Type: application/json" \
    -d '{"id": "1", "name": "Item1"}' \
    https://<API_ID>.execute-api.<REGION>.amazonaws.com/prod/items
```





