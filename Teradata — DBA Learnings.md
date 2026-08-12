
Below is a **combined, expanded 10-lesson Teradata DBA course**, structured like a real production DBA training program rather than just a list of commands.

> **Important:** Teradata administration is version/platform dependent. 
  
  Commands and procedures can differ between Teradata Vantage releases, operating environments, and platforms such as IntelliFlex, VMware, and cloud deployments. 
  
  Treat the commands below as learning examples and always validate them against your specific release documentation before using them in production.

# Teradata Database Administration — Complete 10-Lesson DBA Course

## What you will learn

By the end of these 10 lessons, you should understand the full Teradata DBA lifecycle:

**Architecture → Installation → Configuration → Security → Backup/Recovery → Performance → Patching → Migration → Upgrade → Daily Production Operations**

---

# Lesson 1 — Teradata Architecture & DBA Fundamentals

## 1.1 What is Teradata?

Teradata is a massively parallel processing (**MPP**) database platform.

Instead of one server doing all the work, Teradata distributes database processing across multiple processing units.

A simplified architecture is:

```text
                    Users / Applications
                            |
                       SQL Request
                            |
                    +---------------+
                    | Parsing Engine |
                    |      (PE)      |
                    +---------------+
                            |
                         BYNET
                  /         |        \
                 /          |         \
             AMP 1        AMP 2      AMP N
              |             |          |
           Storage       Storage    Storage
```

The major components you need to understand as a DBA are:

### Parsing Engine — PE

The Parsing Engine is responsible for:

* Receiving SQL
* Parsing SQL
* Checking syntax
* Checking privileges
* Creating execution plans
* Sending work to AMPs
* Returning results

Think of the PE as the **SQL coordinator**.

---

## 1.2 AMP — Access Module Processor

AMPs perform the actual database processing.

An AMP is responsible for:

* Storing rows
* Retrieving rows
* Updating rows
* Joining data
* Aggregating data
* Managing AMP-local storage

The key Teradata DBA concept is:

> **Data distribution across AMPs determines performance.**

If data is distributed evenly, processing can happen in parallel.

If one AMP receives significantly more data/work than others, you have **skew**.

---

# 1.3 BYNET

BYNET provides communication between processing components.

It enables parallel processing and communication between nodes/AMPs.

For a DBA, you don't normally administer BYNET through SQL every day, but you need to understand its role when troubleshooting:

* Node problems
* AMP communication
* Performance problems
* System health

---

# 1.4 Nodes

A node is a physical or virtual processing unit containing Teradata processing resources.

Multiple nodes form a Teradata system.

```text
Teradata System
│
├── Node 1
│   ├── PE
│   └── AMPs
│
├── Node 2
│   ├── PE
│   └── AMPs
│
└── Node N
    ├── PE
    └── AMPs
```

---

# 1.5 Vprocs

A **virtual processor (vproc)** represents a logical processing component.

Important vproc concepts include:

* PE vprocs
* AMP vprocs
* Other system-related vprocs

You will encounter vproc terminology frequently while troubleshooting.

---

# 1.6 Primary Index

The Primary Index (**PI**) is one of the most important Teradata concepts.

Example:

```sql
CREATE TABLE employee
(
    employee_id INTEGER,
    employee_name VARCHAR(100),
    department_id INTEGER
)
PRIMARY INDEX (employee_id);
```

Teradata hashes the PI value to determine where the row should reside.

Conceptually:

```text
employee_id
     |
     v
Hashing Algorithm
     |
     v
Hash Bucket
     |
     v
AMP
```

A good PI generally distributes rows evenly.

---

# 1.7 Data Skew

Suppose you have 10 AMPs.

Ideally:

```text
AMP 1  -> 10%
AMP 2  -> 10%
AMP 3  -> 10%
...
AMP 10 -> 10%
```

Bad distribution might look like:

```text
AMP 1  -> 2%
AMP 2  -> 3%
AMP 3  -> 4%
AMP 4  -> 5%
AMP 5  -> 6%
AMP 6  -> 8%
AMP 7  -> 10%
AMP 8  -> 12%
AMP 9  -> 15%
AMP 10 -> 35%
```

AMP 10 becomes a bottleneck.

---

# 1.8 Fallback

Fallback provides additional protection against AMP failures by maintaining an additional copy of data.

You should understand:

* Fallback protection
* Permanent journal concepts
* Transient journal
* Recovery mechanisms

---

## Lesson 1 Practical Lab

Create a database and table:

```sql
CREATE DATABASE training_db
AS
PERM = 100000000;
```

Then:

```sql
CREATE TABLE training_db.employee
(
    employee_id INTEGER,
    employee_name VARCHAR(100),
    department_id INTEGER
)
PRIMARY INDEX (employee_id);
```

Check the table:

```sql
HELP TABLE training_db.employee;
```

### DBA skills to master

* PE
* AMP
* BYNET
* Node
* Vproc
* PI
* Data distribution
* Skew
* Fallback

---

# Lesson 2 — Teradata Installation & Initial Setup

This lesson covers **installation from a DBA perspective**.

## 2.1 Understand Your Deployment

Before installing anything, determine:

### Platform

Examples include:

* Teradata IntelliFlex
* Teradata Vantage
* Cloud-based Teradata deployments
* Virtualized environments
* Teradata Express/lab environments

The exact installation process differs considerably by platform.

---

# 2.2 Installation Planning

Before installation, collect:

### Hardware

* CPU
* RAM
* Disk
* Network
* Storage capacity

### Operating environment

* Supported OS
* Required packages
* Kernel requirements
* File systems
* Network configuration

### Teradata requirements

* Teradata version
* Database release
* Client software
* Tools
* Licensing
* Deployment architecture

---

# 2.3 Installation Lifecycle

A production installation generally follows this pattern:

```text
Requirement Analysis
        |
        v
Hardware / VM Preparation
        |
        v
Network Configuration
        |
        v
OS Preparation
        |
        v
Teradata Software Installation
        |
        v
Database Initialization
        |
        v
Configuration
        |
        v
Security Setup
        |
        v
Validation
        |
        v
Application Connectivity
```

---

# 2.4 Post-Installation Validation

After installation, verify:

### Database version

```sql
SELECT *
FROM DBC.DBCInfo;
```

### Database objects

```sql
HELP DATABASE DBC;
```

### System health

Use the appropriate system-management tools for your Teradata release to verify:

* Nodes
* AMPs
* PEs
* PDE
* DBS state

---

# 2.5 Client Connectivity

A DBA should understand:

* BTEQ
* Teradata Tools and Utilities
* JDBC
* ODBC
* CLI
* Teradata Parallel Transporter
* Application connectivity

Example BTEQ:

```text
.LOGON hostname/user,password;

SELECT DATE;

.LOGOFF;
```

Never put real production passwords into scripts or command histories.

---

# 2.6 Installation Validation Checklist

Before declaring installation complete:

* [ ] Database starts successfully
* [ ] All nodes healthy
* [ ] AMPs online
* [ ] PE available
* [ ] Client connectivity tested
* [ ] Administrative user created
* [ ] Space configured
* [ ] Security configured
* [ ] Monitoring configured
* [ ] Backup strategy established
* [ ] Documentation completed

---

# Lesson 3 — Database Configuration & User Administration

This is where you begin doing real DBA work.

---

# 3.1 Database Space

Teradata uses different types of space concepts.

You need to understand:

* PERM
* SPOOL
* TEMP
* Database/user allocation
* Space limits

Example:

```sql
CREATE DATABASE sales_db
AS
PERM = 5000000000;
```

---

# 3.2 Users

Example:

```sql
CREATE USER app_user
FROM sales_db
AS
PERM = 1000000000
PASSWORD = 'ExamplePassword';
```

For production, use your organization's approved authentication/password mechanism rather than embedding passwords in scripts.

---

# 3.3 Profiles

Profiles allow common resource/security attributes to be applied to groups of users.

Conceptually:

```text
Profile
   |
   +---- Password rules
   +---- Spool
   +---- Resource settings
   +---- Other user attributes
```

---

# 3.4 Roles

Use roles instead of individually granting hundreds of privileges.

```sql
CREATE ROLE reporting_role;
```

Then grant privileges:

```sql
GRANT SELECT
ON sales_db
TO reporting_role;
```

Then assign the role to users.

```sql
GRANT reporting_role
TO app_user;
```

---

# 3.5 Database Privileges

Important privileges include:

* SELECT
* INSERT
* UPDATE
* DELETE
* EXECUTE
* CREATE
* DROP
* ALTER

Example:

```sql
GRANT SELECT
ON sales_db.customer
TO reporting_role;
```

---

# 3.6 Space Monitoring

DBAs need to monitor:

* Permanent space
* Spool
* Temporary space
* User space
* Database space

Useful system views include appropriate `DBC` space/accounting views for your release.

---

# 3.7 DBA User Lifecycle

Production process:

```text
Request
  ↓
Approval
  ↓
User Creation
  ↓
Profile Assignment
  ↓
Role Assignment
  ↓
Access Validation
  ↓
Audit
```

---

# Lesson 3 Lab

Create:

```text
Database
   ↓
Role
   ↓
User
   ↓
Privileges
```

Then verify:

* User can connect
* User can perform authorized operations
* User cannot access unauthorized objects

This is a very important DBA skill.

---

# Lesson 4 — Security, Auditing & Access Management

A production DBA must treat security as a core responsibility.

---

# 4.1 Authentication

Understand:

* Teradata native authentication
* LDAP
* Kerberos
* External authentication mechanisms
* Single sign-on where applicable

---

# 4.2 Authorization

Authentication answers:

> "Who are you?"

Authorization answers:

> "What are you allowed to do?"

---

# 4.3 Role-Based Access Control

Recommended model:

```text
User
  |
  v
Role
  |
  v
Privileges
  |
  v
Database Objects
```

Avoid unnecessarily granting powerful privileges directly to individual users.

---

# 4.4 Auditing

You should understand:

* DBQL
* Access logging
* Security logging
* User activity
* Failed access
* Privilege changes

DBQL is particularly important for operational and performance analysis.

---

# 4.5 Security DBA Tasks

Regularly review:

* Dormant accounts
* Privileged users
* Roles
* Direct grants
* Failed logins
* Password policies
* Audit configuration
* Service accounts

---

# 4.6 Security Incident Example

Suppose an application suddenly receives:

```text
Access denied
```

Your troubleshooting sequence should be:

```text
User
 ↓
Authentication
 ↓
Role
 ↓
Privilege
 ↓
Object
 ↓
Database
 ↓
Logging
```

Do not immediately grant `ALL` privileges as a workaround.

---

# Lesson 5 — Backup, Restore & Disaster Recovery

This is one of the most important DBA lessons.

A DBA who cannot recover data safely cannot successfully operate a production database.

---

# 5.1 Backup Strategy

Understand:

### Full backup

Complete backup of required data.

### Incremental backup

Backs up changed data according to the mechanism being used.

### Archive

Moves/copies database data for retention or migration purposes.

---

# 5.2 Teradata Backup Ecosystem

Depending on environment, you may encounter:

* Teradata DSA
* BAR
* ARCMAIN
* TPT
* External backup/storage infrastructure

The exact toolset depends on the Teradata version and deployment.

---

# 5.3 Backup Lifecycle

```text
Backup Request
      ↓
Pre-check
      ↓
Backup
      ↓
Validation
      ↓
Catalog/Metadata
      ↓
Retention
      ↓
Restore Test
```

---

# 5.4 Restore Is More Important Than Backup

A backup that has never been restored successfully is not sufficient evidence of recoverability.

Perform restore testing.

Test scenarios such as:

* Table restore
* Database restore
* User restore
* Permission recovery
* Disaster recovery

---

# 5.5 Recovery Objectives

Understand:

### RPO

**Recovery Point Objective**

How much data loss can the organization tolerate?

### RTO

**Recovery Time Objective**

How quickly must the service be restored?

Example:

```text
RPO = 15 minutes
RTO = 2 hours
```

Your backup/DR architecture must support those objectives.

---

# 5.6 Disaster Recovery

Understand:

* Primary system
* DR system
* Replication
* Backup storage
* Network dependency
* DNS/application failover
* Recovery procedures
* Validation

---

# Lesson 5 Lab

Build a recovery exercise:

```text
Create table
      ↓
Insert data
      ↓
Backup
      ↓
Delete/change data
      ↓
Restore
      ↓
Validate row counts
      ↓
Validate application access
```

---

# Lesson 6 — Performance Monitoring & Troubleshooting

This is where a DBA moves from basic administration to advanced administration.

---

# 6.1 Performance Areas

Monitor:

* CPU
* I/O
* AMP utilization
* Skew
* Spool
* Query duration
* Concurrent queries
* Locking
* Parsing
* Network activity
* Workload performance

---

# 6.2 EXPLAIN

One of the most important tools is:

```sql
EXPLAIN
SELECT *
FROM sales_db.customer
WHERE customer_id = 1001;
```

Learn to interpret:

* AMP steps
* Row estimates
* Joins
* Redistributions
* Duplications
* Full-table scans
* Confidence
* Spool

---

# 6.3 Statistics

Statistics help the optimizer estimate data distribution and choose better execution plans.

Example:

```sql
COLLECT STATISTICS
ON sales_db.customer
COLUMN customer_id;
```

You need to understand:

* When to collect statistics
* Which columns need statistics
* Multi-column statistics
* Statistics freshness
* Statistics after major data changes

---

# 6.4 Skew

A DBA should regularly investigate skew.

Common causes:

* Poor PI
* Low-cardinality PI
* Null-heavy PI
* Bad data distribution
* Data model issues

---

# 6.5 Spool Problems

A common production problem:

```text
2646: No more spool space
```

Possible causes include:

* Huge intermediate result
* Bad join
* Cartesian product
* Poor statistics
* Data skew
* Insufficient spool
* Poor SQL design

Don't automatically increase spool.

First determine **why the query needs excessive spool**.

---

# 6.6 Locking

Learn:

* Row/hash locks
* Table locks
* Blocking
* Deadlocks
* Transaction behavior

A production DBA should be able to answer:

> Which session is blocking my application?

---

# 6.7 DBQL

DBQL can help answer:

* Who ran the query?
* How long did it run?
* How much CPU did it consume?
* How much I/O did it use?
* How much spool did it use?
* When did it run?

---

# 6.8 Performance Troubleshooting Method

Use:

```text
Problem
  ↓
Identify Query
  ↓
Check DBQL
  ↓
Check EXPLAIN
  ↓
Check Statistics
  ↓
Check PI
  ↓
Check Skew
  ↓
Check Locks
  ↓
Check CPU/I/O
  ↓
Fix
  ↓
Re-test
```

---

# Lesson 7 — Patching & Routine Maintenance

Now we move into **production lifecycle management**.

---

# 7.1 Why Patch?

Patches may address:

* Security vulnerabilities
* Software defects
* Stability issues
* Performance issues
* Compatibility
* Hardware/OS support

---

# 7.2 Patch Lifecycle

Never treat patching as:

```text
Download → Install
```

Use:

```text
Requirement
   ↓
Version Assessment
   ↓
Release Notes
   ↓
Compatibility Check
   ↓
Change Request
   ↓
Backup
   ↓
Test Environment
   ↓
Validation
   ↓
Production
   ↓
Post-check
   ↓
Monitoring
```

---

# 7.3 Pre-Patching Checklist

* [ ] Confirm target release
* [ ] Review release notes
* [ ] Review known issues
* [ ] Verify supported configuration
* [ ] Check application compatibility
* [ ] Verify backup
* [ ] Confirm rollback strategy
* [ ] Notify stakeholders
* [ ] Schedule maintenance window
* [ ] Confirm monitoring availability

---

# 7.4 During Patching

Monitor:

* Node status
* AMP status
* Database status
* System alerts
* Application connectivity
* Errors
* Resource utilization

Follow the exact vendor procedure for your environment.

---

# 7.5 Post-Patch Validation

Check:

```text
Database
   ↓
Nodes
   ↓
AMPs
   ↓
PEs
   ↓
Users
   ↓
Applications
   ↓
Queries
   ↓
Backup
   ↓
Monitoring
```

---

# 7.6 Routine Maintenance

### Daily

* Check system health
* Check failed queries
* Check space
* Check active sessions
* Check critical workloads
* Review alerts

### Weekly

* Review performance
* Review skew
* Review backups
* Review failed jobs
* Review storage trends

### Monthly

* Capacity planning
* Security review
* User review
* Patch review
* Statistics review
* DR readiness

---

# Lesson 8 — Teradata Migration

Migration means moving data and/or workloads between Teradata environments.

Examples:

```text
DEV → TEST
TEST → UAT
UAT → PROD
OLD → NEW
ON-PREM → CLOUD
```

---

# 8.1 Migration Components

A complete migration may include:

### Database objects

* Databases
* Tables
* Views
* Macros
* Stored procedures
* Functions

### Data

* Table data
* Historical data

### Security

* Users
* Roles
* Privileges
* Profiles

### Operational configuration

* Jobs
* Monitoring
* Backup
* Workloads
* Connections

---

# 8.2 Migration Tools

Depending on requirements:

* Teradata Parallel Transporter
* BTEQ
* ARC/DSA
* SQL-based migration
* QueryGrid
* External ETL tools

---

# 8.3 Migration Strategy

A good migration plan looks like:

```text
Discovery
   ↓
Inventory
   ↓
Dependency Analysis
   ↓
Schema Migration
   ↓
Security Migration
   ↓
Data Migration
   ↓
Validation
   ↓
Performance Testing
   ↓
Application Testing
   ↓
Cutover
   ↓
Post-Cutover Monitoring
```

---

# 8.4 Data Validation

Never assume:

> "The migration completed successfully, therefore the data is correct."

Validate:

### Row counts

```sql
SELECT COUNT(*)
FROM source_db.customer;
```

Compare against target:

```sql
SELECT COUNT(*)
FROM target_db.customer;
```

Also validate:

* Aggregates
* NULL counts
* Duplicate counts
* Min/max values
* Business-critical records
* Referential relationships
* Application results

---

# 8.5 Migration Cutover

A production cutover should have:

### Before

* Freeze/change control
* Final backup
* Final data sync
* Validation

### During

* Stop writes
* Perform final migration
* Validate
* Switch applications

### After

* Monitor
* Validate application
* Validate performance
* Keep rollback capability

---

# Lesson 9 — Teradata Upgrade

Patching and upgrading are related but not identical.

### Patch

Usually addresses fixes/security/stability within a supported release path.

### Upgrade

Moves the system to a newer software release/version.

---

# 9.1 Upgrade Planning

Before upgrading:

### Inventory

Identify:

* Current version
* Target version
* Applications
* Drivers
* ETL tools
* Utilities
* Stored procedures
* External integrations

---

# 9.2 Compatibility Assessment

Check:

* SQL compatibility
* Client compatibility
* Driver compatibility
* Application compatibility
* Utility compatibility
* Authentication
* Workload management
* Backup/restore
* Monitoring

---

# 9.3 Upgrade Lifecycle

```text
Current State
     ↓
Assessment
     ↓
Compatibility
     ↓
Test Upgrade
     ↓
Application Testing
     ↓
Performance Testing
     ↓
Rollback Planning
     ↓
Production Change
     ↓
Upgrade
     ↓
Validation
     ↓
Monitoring
```

---

# 9.4 Pre-Upgrade Checklist

* [ ] Current version documented
* [ ] Target version approved
* [ ] Compatibility reviewed
* [ ] Application inventory complete
* [ ] Backup verified
* [ ] DR plan reviewed
* [ ] Rollback plan documented
* [ ] Test upgrade completed
* [ ] Performance baseline captured
* [ ] Maintenance window approved
* [ ] Stakeholders notified

---

# 9.5 Post-Upgrade Validation

Validate:

### Database

* Version
* Nodes
* AMPs
* PEs
* System health

### Security

* Users
* Roles
* Authentication
* Privileges

### Applications

* JDBC
* ODBC
* ETL
* BI tools

### Performance

Compare against your pre-upgrade baseline.

---

# Lesson 10 — Production DBA Operations & Troubleshooting

This final lesson combines everything into real-world DBA work.

---

# 10.1 Daily DBA Routine

Start the day with:

```text
System Health
     ↓
Space
     ↓
Sessions
     ↓
Queries
     ↓
Locks
     ↓
CPU/I/O
     ↓
Alerts
     ↓
Backup Status
```

---

# 10.2 Daily Health Checks

Check:

### System

* Nodes
* AMPs
* PDE
* DBS state

### Storage

* Permanent space
* Spool
* Temporary space
* Disk utilization

### Workload

* Long-running queries
* High CPU queries
* High I/O queries
* Failed queries

### Sessions

* Active sessions
* Suspicious sessions
* Blocking sessions

---

# 10.3 Common Production Problems

## Problem 1 — No Spool Space

Possible causes:

```text
Bad Join
   +
Poor Statistics
   +
Data Skew
   +
Large Intermediate Result
```

Troubleshooting:

1. Identify query.
2. Run EXPLAIN.
3. Check statistics.
4. Check PI.
5. Check skew.
6. Check spool usage.
7. Optimize SQL.
8. Re-test.

---

# Problem 2 — Query Suddenly Became Slow

Investigate:

```text
Query History
     ↓
DBQL
     ↓
EXPLAIN
     ↓
Statistics
     ↓
Data Volume
     ↓
PI
     ↓
Skew
     ↓
Locks
     ↓
System CPU/I/O
```

Possible causes:

* Statistics changed
* Data volume increased
* Execution plan changed
* Blocking
* System workload increased
* Skew
* Resource allocation

---

# Problem 3 — User Cannot Access Table

Check:

```text
Can user authenticate?
        ↓
Is user active?
        ↓
Does user have role?
        ↓
Does role have privilege?
        ↓
Is privilege on correct object?
        ↓
Is database/object correct?
```

---

# Problem 4 — Application Cannot Connect

Check:

```text
Application
    ↓
Network
    ↓
Port
    ↓
DNS
    ↓
Driver
    ↓
Authentication
    ↓
Teradata session
    ↓
User permissions
```

Don't assume the database is the problem.

---

# Problem 5 — Database Space Is Increasing

Investigate:

* Largest databases
* Largest tables
* Historical growth
* Unused tables
* Temporary data
* Backup/archive strategy
* Data retention
* Application changes

Then create a capacity forecast.

---

# 10.4 Incident Management

A strong DBA doesn't just fix the immediate problem.

Use:

```text
Incident
   ↓
Immediate Recovery
   ↓
Root Cause Analysis
   ↓
Permanent Fix
   ↓
Monitoring
   ↓
Documentation
   ↓
Prevention
```

---

# 10.5 DBA Documentation

Maintain:

### Environment document

```text
Teradata Version
Nodes
AMPs
PEs
Storage
Applications
Connections
```

### Security document

```text
Users
Roles
Profiles
Authentication
Privileged Accounts
```

### Backup document

```text
Backup Tool
Schedule
Retention
Storage
Restore Procedure
DR
```

### Upgrade document

```text
Current Version
Target Version
Compatibility
Testing
Rollback
Cutover
Validation
```

---

# Complete Teradata DBA Skill Matrix

| Area          | Beginner      | Intermediate     | Advanced              |
| ------------- | ------------- | ---------------- | --------------------- |
| Architecture  | PE/AMP        | BYNET/Vproc      | MPP internals         |
| Installation  | Basic         | Production setup | Enterprise deployment |
| Configuration | Users/DBs     | Profiles/Roles   | System optimization   |
| Security      | GRANT         | RBAC/Auditing    | Enterprise security   |
| Backup        | Basic backup  | Restore          | DR                    |
| Performance   | EXPLAIN       | Stats/Skew       | Workload tuning       |
| Patching      | Understand    | Execute          | Plan enterprise patch |
| Migration     | Tables        | Databases        | Full environment      |
| Upgrade       | Concepts      | Execute          | Plan/rollback         |
| Operations    | Health checks | Troubleshooting  | Production ownership  |

---

# The 10-Lesson Hands-On Project

Instead of learning the lessons independently, use **one continuous project**.

## Project: Build and Operate a Production-Style Teradata Environment

### Phase 1 — Build

```text
Teradata Environment
       ↓
Database
       ↓
Users
       ↓
Roles
       ↓
Tables
       ↓
Data
```

### Phase 2 — Secure

```text
Authentication
       ↓
Roles
       ↓
Privileges
       ↓
Auditing
```

### Phase 3 — Operate

```text
Monitoring
       ↓
Space
       ↓
Sessions
       ↓
Queries
       ↓
Locks
```

### Phase 4 — Optimize

```text
EXPLAIN
   ↓
Statistics
   ↓
PI
   ↓
Skew
   ↓
Performance
```

### Phase 5 — Protect

```text
Backup
   ↓
Restore
   ↓
DR
```

### Phase 6 — Maintain

```text
Patch
   ↓
Validate
   ↓
Monitor
```

### Phase 7 — Migrate

```text
Source
  ↓
Extract
  ↓
Transfer
  ↓
Load
  ↓
Validate
```

### Phase 8 — Upgrade

```text
Assessment
   ↓
Testing
   ↓
Backup
   ↓
Upgrade
   ↓
Validation
   ↓
Monitoring
```

---

# Your 10-Week DBA Training Schedule

| Week | Lesson          | Main Skill         | Hands-On            |
| ---- | --------------- | ------------------ | ------------------- |
| 1    | Architecture    | PE/AMP/BYNET/PI    | Create DB & tables  |
| 2    | Installation    | Build environment  | Install/connect     |
| 3    | Configuration   | Users/Roles/Space  | User administration |
| 4    | Security        | RBAC/Auditing      | Access control      |
| 5    | Backup/Recovery | Protect data       | Backup/restore      |
| 6    | Performance     | EXPLAIN/Stats/Skew | Tune queries        |
| 7    | Patching        | Maintenance        | Patch simulation    |
| 8    | Migration       | Move data          | DEV → TEST          |
| 9    | Upgrade         | Version lifecycle  | Upgrade simulation  |
| 10   | Operations      | Production DBA     | Incident simulation |

---

# Most Important DBA Commands/Concepts to Master

Don't try to memorize hundreds of commands initially. Master these categories first:

```text
DBC
 ├── Users
 ├── Roles
 ├── Profiles
 ├── Space
 ├── Sessions
 ├── Queries
 ├── DBQL
 └── System Information

SQL
 ├── CREATE
 ├── ALTER
 ├── DROP
 ├── GRANT
 ├── REVOKE
 ├── COLLECT STATISTICS
 └── EXPLAIN

Administration
 ├── Backup
 ├── Restore
 ├── Patch
 ├── Upgrade
 ├── Migration
 ├── Monitoring
 └── Recovery
```

## The mindset of a good Teradata DBA

Don't become a DBA who only memorizes commands.

For every production problem, think:

**What changed? → Where is the problem? → What evidence do I have? → What is the impact? → What is the safest fix? → How do I validate it? → How do I prevent it from happening again?**

That mindset is what takes you from **Teradata SQL user → Teradata DBA → Production Teradata DBA**.
