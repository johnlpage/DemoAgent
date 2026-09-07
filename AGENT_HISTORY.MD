# Agent History

Tracking progress through the Memex MVP runbook (README.md).

## Prerequisites
- Java 21 (Homebrew) - OK
- Maven 3.9.14 - OK
- Python 3.14.3 - OK
- Git 2.50.1 - OK
- Docker 29.7.2 - OK
- Atlas Local: already running as container `AtlasLocal`, `127.0.0.1:27055` - decided to reuse existing instance (user's choice)

## Step 1: Creating Sample Data
- User has no customer sample data.
- Use case: Bank customer service complaints.
- Modeled example.json on the CFPB Consumer Complaint Database structure (complaintId, dateReceived, product/subProduct, issue/subIssue, consumerComplaintNarrative, company response fields, consumer info, state/zip, etc.)
- User asked to add an array capturing processing history / customer contacts -> added `processingHistory` array (eventDate, eventType, actor, channel, notes) with 6 sample lifecycle events (Received -> Assigned -> Customer Contact -> Escalation -> Customer Contact -> Closed).
- User confirmed ("yes") the shape is good.
- Saved to `example.json` in this directory.

## Step 2: Generating Sample Data with DataGen
- Status: IN PROGRESS
- Next: clone MongoEnterpriseMicroserviceExamples repo (shallow, then remove .git), explore DataGen subdirectory, build it, read its README for how to generate a statistical model from example.json (single doc, so need to research field value distributions ourselves since we don't have 200+ real docs).

## Stage 3: Building Memex
- Status: NOT STARTED

## Stage 4: Loading Sample Data
- Status: NOT STARTED

## Stage 5: Custom Endpoints
- Status: NOT STARTED

## Stage 6: Cloud Deployment
- Status: NOT STARTED

## Step 7: Final Usage Docs
- Status: NOT STARTED
