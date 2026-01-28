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

<img width="1366" height="768" alt="createtable" src="https://github.com/user-attachments/assets/9e1ea5d9-e78d-4355-b5f8-d1ebd02fd872" />

<img width="1366" height="768" alt="readcapacity" src="https://github.com/user-attachments/assets/040dd4ee-85cd-439b-ad07-da85bb5e5d2e" />

<img width="1366" height="768" alt="attribute added" src="https://github.com/user-attachments/assets/a685a3e9-f3fd-406d-a707-684f7c1651f8" />



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

<img width="1366" height="768" alt="createtablesusingcli" src="https://github.com/user-attachments/assets/ddfc6cc6-4d09-4ce9-a769-f59ce3fe28d0" />


Verified tables were created:

```bash
aws dynamodb wait table-exists --table-name ContentCatalog
aws dynamodb wait table-exists --table-name Forum
aws dynamodb wait table-exists --table-name Post
aws dynamodb wait table-exists --table-name Comment
```

<img width="1366" height="768" alt="tables created" src="https://github.com/user-attachments/assets/510d9453-9aac-488c-b14a-be8da5c566dd" />

## Step 3: Load Data

Downloaded sample data and loaded it into tables:

```bash
curl -O https://storage.googleapis.com/nextwork_course_resources/courses/aws/AWS%20Project%20People%20projects/Project%3A%20Query%20Data%20with%20DynamoDB/nextworksampledata.zip
unzip nextworksampledata.zip
cd nextworksampledata
```
<img width="1366" height="768" alt="unzipfile" src="https://github.com/user-attachments/assets/71b68f8b-acd6-4752-9d4e-02425f8432ef" />

Loaded data using batch-write-item:

```bash
aws dynamodb batch-write-item --request-items file://ContentCatalog.json
aws dynamodb batch-write-item --request-items file://Forum.json
aws dynamodb batch-write-item --request-items file://Post.json
aws dynamodb batch-write-item --request-items file://Comment.json
```

<img width="1366" height="768" alt="more data added" src="https://github.com/user-attachments/assets/5dee18ec-ac3f-4a0c-8611-8bf80cbbc9f0" />

## Step 4: Explore Table Structure

Examined the `ContentCatalog` table and discovered:
- 6 items with different attribute sets
- Mix of Projects and Videos content types
- Each item has its own unique attributes (DynamoDB's flexible schema in action)

Added a new attribute `StudentsComplete` with value "Sharon" to one Project item, demonstrating how DynamoDB allows different items to have different attributes.

<img width="1366" height="768" alt="sharonatt" src="https://github.com/user-attachments/assets/1c7adf37-28a3-43f6-8167-a371e02a3679" />

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
