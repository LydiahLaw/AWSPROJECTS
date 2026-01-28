# DynamoDB Tables Project

A hands-on project for creating and managing DynamoDB tables using AWS Console and CloudShell.

## What I Built

- Created DynamoDB tables through AWS Console
- Used AWS CloudShell and CLI to automate table creation
- Loaded sample data into multiple tables
- Explored DynamoDB's flexible schema structure

## Architecture

This project uses:
- **Amazon DynamoDB** - NoSQL database service
- **AWS CloudShell** - Browser-based shell with AWS CLI pre-installed
- **AWS CLI** - Command line tool for managing AWS resources

## Step 1: Create First Table (Console)

Created a table called `AWSstudents` with:
- **Table name:** `AWSstudents`
- **Partition key:** `StudentName` (String)
- **Read/Write capacity:** 1 RCU, 1 WCU (provisioned)
- **Auto scaling:** Disabled (to stay within Free Tier)

Added first student:
- **StudentName:** Sharon
- **ProjectsComplete:** 4

![AWSstudents table screenshot]

## Step 2: Create Tables with CloudShell

Used AWS CloudShell to create four additional tables with a single command:

```bash
aws dynamodb create-table \
    --table-name ContentCatalog \
    --attribute-definitions \
        AttributeName=Id,AttributeType=N \
    --key-schema \
        AttributeName=Id,KeyType=HASH \
    --provisioned-throughput \
        ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --query "TableDescription.TableStatus"

aws dynamodb create-table \
    --table-name Forum \
    --attribute-definitions \
        AttributeName=Name,AttributeType=S \
    --key-schema \
        AttributeName=Name,KeyType=HASH \
    --provisioned-throughput \
        ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --query "TableDescription.TableStatus"

aws dynamodb create-table \
    --table-name Post \
    --attribute-definitions \
        AttributeName=ForumName,AttributeType=S \
        AttributeName=Subject,AttributeType=S \
    --key-schema \
        AttributeName=ForumName,KeyType=HASH \
        AttributeName=Subject,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --query "TableDescription.TableStatus"

aws dynamodb create-table \
    --table-name Comment \
    --attribute-definitions \
        AttributeName=Id,AttributeType=S \
        AttributeName=CommentDateTime,AttributeType=S \
    --key-schema \
        AttributeName=Id,KeyType=HASH \
        AttributeName=CommentDateTime,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --query "TableDescription.TableStatus"
```

Verified tables were created:

```bash
aws dynamodb wait table-exists --table-name ContentCatalog
aws dynamodb wait table-exists --table-name Forum
aws dynamodb wait table-exists --table-name Post
aws dynamodb wait table-exists --table-name Comment
```

![CloudShell table creation screenshot]

## Step 3: Load Data

Downloaded sample data and loaded it into tables:

```bash
curl -O https://storage.googleapis.com/nextwork_course_resources/courses/aws/AWS%20Project%20People%20projects/Project%3A%20Query%20Data%20with%20DynamoDB/nextworksampledata.zip
unzip nextworksampledata.zip
cd nextworksampledata
```

Loaded data using batch-write-item:

```bash
aws dynamodb batch-write-item --request-items file://ContentCatalog.json
aws dynamodb batch-write-item --request-items file://Forum.json
aws dynamodb batch-write-item --request-items file://Post.json
aws dynamodb batch-write-item --request-items file://Comment.json
```

![Data loading screenshot]

## Step 4: Explore Table Structure

Examined the `ContentCatalog` table and discovered:
- 6 items with different attribute sets
- Mix of Projects and Videos content types
- Each item has its own unique attributes (DynamoDB's flexible schema in action)

Added a new attribute `StudentsComplete` with value "Sharon" to one Project item, demonstrating how DynamoDB allows different items to have different attributes.

![ContentCatalog table screenshot]

## Key Learnings

**DynamoDB vs Relational Databases:**
- DynamoDB items can have different attributes (flexible schema)
- Relational databases require all rows to have the same columns
- DynamoDB uses partition keys for fast data retrieval
- Better for varying data structures and high-speed lookups

**Capacity Units:**
- Read Capacity Units (RCUs): 1 RCU = 2 item reads/second
- Write Capacity Units (WCUs): 1 WCU = 1 item write/second
- AWS Free Tier includes 25 RCUs and 25 WCUs

**CLI Benefits:**
- Faster than console for bulk operations
- Easily repeatable and scriptable
- Essential for automation

## Cleanup

Deleted all resources to avoid charges:

```bash
aws dynamodb delete-table --table-name Comment
aws dynamodb delete-table --table-name Forum
aws dynamodb delete-table --table-name ContentCatalog
aws dynamodb delete-table --table-name Post
aws dynamodb delete-table --table-name AWSstudents
```

## Tools Used

- AWS Management Console
- AWS CloudShell
- AWS CLI
- Amazon DynamoDB

## Cost

$0 - Stayed within AWS Free Tier limits
