# Case Study: Designing a File Sharing System Similar to OneDrive

## Context

Your company wants to develop a platform that allows users to:

- Upload files
- Access their files from any device
- Share files with other users
- Synchronize changes in real time

The system must be able to support several million users.

---

## 1. Requirements Gathering

### Functional Requirements

The system must allow users to:

- Create a user account
- Upload a file
- Upload multiple versions of the same file
- Delete a file
- Share a file through a link
- Synchronize files across devices

### Non-Functional Requirements

- High availability (>99.9%)
- Low response time
- Data security
- Horizontal scalability
- Very large storage capacity

---

## 2. Capacity Estimation

**Assumptions:**

- 10 million users
- 1 million daily active users
- 100 files per user
- Average file size: 5 MB

**Storage volume:**

```
10,000,000 × 100 × 5 MB
≈ 5,000 TB
≈ 5 PB
```

The system must handle several petabytes of data.

---

## 3. High-Level Architecture

```
+------------------+
| Applications     |
| Web / Mobile     |
+--------+---------+
         |
         v
+------------------+
| API Gateway      |
+--------+---------+
         |
+--------+------------------+
|                           |
v                           v
+--------------------+   +------------------+
| Metadata Service   |   | Sharing Service  |
+--------+-----------+   +------------------+
         |
         v
+------------------+
| SQL Database     |
+------------------+
         |
         v
+------------------+
| Object Storage   |
| (Blob Storage)   |
+------------------+
```

---

## 4. Main Components

### API Gateway

**Responsibilities:**

- Authentication
- Authorization
- Rate limiting
- Request routing

### Metadata Service

**Stores:**

- File name
- File size
- Creation date
- Owner
- Physical file location

**Example:**

```json
{
  "fileId": "12345",
  "userId": "789",
  "fileName": "contract.pdf",
  "size": 5242880,
  "storagePath": "/blob/ab/cd/file"
}
```

### Database

A relational database is a good fit:

- PostgreSQL
- Azure SQL
- SQL Server

**Tables:**

```
Users
Files
Shares
Versions
```

### Object Storage

The files themselves are not stored in the database.

**Examples:**

- Azure Blob Storage
- Amazon S3
- Google Cloud Storage

**Benefits:**

- Low cost
- High durability
- Virtually unlimited scalability

---

## 5. Upload Management

**Process:**

```
Client
  |
  | Upload
  v
API
  |
  | Generates secure URL
  v
Blob Storage
```

**Steps:**

1. The user requests an upload.
2. The API generates a temporary URL.
3. The client uploads the file directly to storage.
4. Metadata is stored in the database.

This approach prevents overloading application servers.

---

## 6. Synchronization

To detect changes:

```
Version 1
Version 2
Version 3
```

**Each modification:**

- Creates a new version
- Updates metadata
- Triggers a notification

**Possible technologies:**

- WebSocket
- SignalR
- Azure Event Grid

---

## 7. File Sharing

A user generates a link:

```
https://my-drive.com/share/abc123
```

**The Share table contains:**

```
ShareId
FileId
ExpirationDate
Permissions
```

**Possible permissions:**

- Read
- Edit
- Download

---

## 8. Scalability

### Stateless Servers

All APIs should be stateless.

```
Load Balancer
      |
   +--+--+
   |     |
 API1   API2
   |     |
 API3   API4
```

This makes it easy to add new servers as demand increases.

### Cache

Use **Redis** to store:

- Sessions
- User information
- Most frequently accessed files

**Benefits:**

- Reduced SQL database load
- Improved response times

---

## 9. Security

### Authentication

- OAuth 2.0
- OpenID Connect
- Microsoft Entra ID

### Encryption

**At Rest:**

```
AES-256
```

**In Transit:**

```
HTTPS / TLS
```

### Access Control

Each request verifies:

```
UserId == OwnerId
```

or

```
User has a sharing permission
```

---

## 10. Monitoring and Observability

**Tools:**

- Azure Monitor
- Application Insights
- Grafana
- Prometheus

**Metrics:**

- Number of uploads
- Response time
- HTTP errors
- Storage capacity
- Active users

---

## Risks and Future Improvements

| Risk                  | Solution                                   |
|-----------------------|--------------------------------------------|
| Storage saturation    | Distribute storage across multiple regions |
| Data loss             | Multi-region replication                   |
| High response times   | CDN + caching                              |
| Heavy load            | Auto-scaling                               |
| Cyberattacks          | WAF and DDoS protection                    |

---

## Conclusion

This design is based on four key principles:

1. Separation of metadata and file content
2. Object storage for file data
3. Stateless architecture for scalability
4. Security and high availability by design
