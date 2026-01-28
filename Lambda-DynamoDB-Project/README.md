# Lambda Function with DynamoDB Integration

Built a serverless function that retrieves user data from a DynamoDB table - no servers to manage!

## What I Built

- Created a DynamoDB table to store user data
- Built a Lambda function using Node.js
- Connected Lambda to DynamoDB using AWS SDK
- Fixed permission issues between Lambda and DynamoDB
- Tightened security with a custom inline policy

## Architecture

**Services used:**
- **Amazon DynamoDB** - NoSQL database for storing user data
- **AWS Lambda** - Serverless function to retrieve data
- **IAM** - Permissions management
- **CloudWatch** - Function logs (automatically set up)

## Step 1: Create DynamoDB Table

Set up a DynamoDB table to hold user information.

**Table configuration:**
- **Table name:** `UserData`
- **Partition key:** `userId` (String)
- **Settings:** Default (no custom capacity settings needed)

![DynamoDB table creation]

Added sample data in JSON format:

```json
{
  "userId": "1",
  "name": "Test User",
  "email": "test@example.com"
}
```

The cool thing about DynamoDB? It's schemaless. You can add any attributes you want without defining them first - super flexible compared to traditional databases.

![Sample user data]

## Step 2: Create Lambda Function

Built a Lambda function to retrieve data from the table.

**Function setup:**
- **Function name:** `RetrieveUserData`
- **Runtime:** Node.js (latest version)
- **Execution role:** Created new role with basic Lambda permissions

![Lambda function creation]

## Step 3: Write the Function Code

Replaced the default code with this:

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand } from "@aws-sdk/lib-dynamodb";

const ddbClient = new DynamoDBClient({ region: 'us-east-1' }); // Updated with actual region
const ddb = DynamoDBDocumentClient.from(ddbClient);

async function handler(event) {
    const userId = String(event.userId);
    const params = {
        TableName: 'UserData',
        Key: { userId }
    };

    try {
        const command = new GetCommand(params);
        const data = await ddb.send(command);
        console.log("DynamoDB Response:", JSON.stringify(data));
        const { Item } = data;
        if (Item) {
            console.log("User data retrieved:", Item);
            return Item;
        } else {
            console.log("No user data found for userId:", userId);
            return null;
        }
    } catch (err) {
        console.error("Unable to retrieve data:", err);
        return err;
    }
}

export { handler };
```

**What this code does:**
- Uses AWS SDK to talk to DynamoDB
- Takes a `userId` as input
- Fetches the matching user from the `UserData` table
- Returns the user data or an error message

Deployed the function and moved on to testing.

![Code deployment]

## Step 4: First Test - Oops, Access Denied!

Created a test event with this JSON:

```json
{
  "userId": "1"
}
```

Ran the test and... got an error!

```
User: arn:aws:sts::123456789:assumed-role/RetrieveUserData-role-xyz/RetrieveUserData 
is not authorized to perform: dynamodb:GetItem on resource: arn:aws:dynamodb:us-east-1:123456789:table/UserData
```

The function worked fine, but Lambda didn't have permission to read from DynamoDB. Classic AWS moment.

![Access denied error]

## Step 5: Fix Permissions

Went to the Lambda function's Configuration → Permissions tab and clicked on the execution role.

In IAM, added the `AmazonDynamoDBReadOnlyAccess` policy:
- Searched for "DynamoDB" in policies
- Selected `AmazonDynamoDBReadOnlyAccess`
- This policy includes the `GetItem` permission we needed

![Attaching DynamoDB policy]

## Step 6: Test Again - Success!

Ran the test again and it worked! Got back the user data:

```json
{
  "userId": "1",
  "name": "Test User",
  "email": "test@example.com"
}
```

![Successful test result]

## Secret Mission: Tighten Security

The `AmazonDynamoDBReadOnlyAccess` policy gives read access to *all* DynamoDB tables in my account. That's more than needed.

Created a custom inline policy that only allows reading from the `UserData` table.

**Steps:**
1. Removed the `AmazonDynamoDBReadOnlyAccess` policy
2. Created a new inline policy using JSON editor
3. Named it `DynamoDB_UserData_ReadAccess`

**Custom policy (JSON):**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DynamoDBReadAccess",
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-east-1:123456789:table/UserData"
            ]
        }
    ]
}
```

Tested the function again - still works perfectly, but now with minimal permissions!

![Custom inline policy]

![Final test success]

## Key Learnings

**About Lambda:**
- Runs code without managing servers
- Only pay for compute time used
- Scales automatically
- Perfect for event-driven applications

**About IAM Permissions:**
- Lambda needs explicit permissions to access other AWS services
- Always use least privilege (only grant what's needed)
- Managed policies are quick but broad
- Inline policies give fine-grained control

**DynamoDB + Lambda Use Cases:**
- User authentication and profile retrieval
- E-commerce product lookups
- Social media content feeds
- Blog/news article fetching
- Any app needing fast data access

## Cleanup

Deleted all resources to avoid charges:

1. **Lambda function:** Actions → Delete → confirmed
2. **DynamoDB table:** Delete → confirmed (kept CloudWatch alarm deletion checked, skipped backup)

## Tools Used

- AWS Lambda
- Amazon DynamoDB
- AWS IAM
- AWS CloudWatch (auto-configured)
- AWS SDK for JavaScript

## Cost

$0 - Everything stayed within Free Tier limits
