# Sales Report Generator

## System Design Document

This document outlines the system design for generating sales reports for a given date range and product group (e.g., Samsung) and the report format will be PowerPoint presentation document.

### Problem Statement

Allow users to generate sales reports based on:

- Date Range
- Product Group (e.g., Samsung)
- Report format is PPTX (PowerPoint)

#### Non-functional requirements

- Support many concurrent users
- Sales data can reside in multiple datastoresR
- Retry mechanism in case if a report generation attempt fails
- Logging

### Tech Stack

**Backend:** .NET Core Web API  
**Database:** SQL Server  
**Queue:** RabbitMQ  
**File Storage:** Azure Blob storage  
**Report Generator:** OpenXML SDK (for PPTX)  

## Approach A: A simple synchronous API-based design

### High-Level flow

[Client] --> [API Controller] --> [Service Layer] --> Sales [Data Aggregator] --> [Report Generator (PPTX)] --> [File System] --> [Return Download URL]

### Step by step flowf

- Client hits `/api/sales/report` with date range and product group
- Controller calls service --> service collects sales data
- Service generates PPTX using OpenXML library
- Generated report uploaded to Azure Blob storage
- URL returned to client

### Flowchart

To be added

### Disadvantages

- Client/user to reissue the request in case of timeout issues or any failures

## Approach B: With RabbitMQ

### High-Level Design

[Client] --> [API Controller] --> [Save requestin Job DB] --> [Add a message to RabbitMQ] --> [Worker Service] --> [Sales Data Aggregator] --> [Report Generator] --> [File Storage] and [Job DB: update status]

### Step by step Flow

- Client calls `/api/sales/report` with request body
- API stores job with status Pending in SQL DB
- API send a request to RabbitMQ with job ID
- Worker service listens to RabbitMQ
- Worker pulls the sales data --> generates PPTX report → uploads file
- Worker updates DB with Completed status and file URL
- Client polls job status or receives notification

### Flowchart

To be added

### Logging

- Use Serilog
- Track job status, errors, etc.,

### Advantages

- Scalable
- No HTTP timeout issues
- Retry feature

### Future enhancements

- Email notification when the report is ready or job failed
- Use Web sockets (SignalR) for realtime job status in the dashboard
- Auto delete the old reports from Blob storage
