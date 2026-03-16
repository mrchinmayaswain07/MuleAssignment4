# MuleSoft API-Led Connectivity Project

## Salesforce Composite Integration

## Overview

This project demonstrates the **API-Led Connectivity architecture using MuleSoft**.
The system integrates external public APIs with Salesforce using a layered architecture consisting of **Experience API, Process API, and System APIs**.

The solution fetches **users and posts from an external API**, transforms the data into a **company–employee structure**, and synchronizes it into **Salesforce Accounts and Contacts using the Salesforce Composite API**.

---

# Architecture

The project follows the **4-layer API-Led architecture**:

Client
↓
Experience API (8081)
↓
Process API (8082)
↓
External Server API (8083)
Salesforce System API (8084)
↓
Salesforce

---

# APIs in the Project

## 1. Experience API

**Port:** 8081

Purpose:

* Exposes the final endpoint to the client.
* Calls the Process API.

Endpoint:
GET /syncCompanies

Flow:
Client → Experience API → Process API

---

## 2. Process API

**Port:** 8082

Purpose:

* Orchestrates the integration.
* Calls External Server API.
* Transforms the data.
* Calls Salesforce System API.

Endpoint:
GET /api/syncCompanies

Process Flow:

1. Fetch users from External API
2. Fetch posts from External API
3. Transform data into Company + Employees structure
4. Convert into Salesforce Composite requests
5. Send batched requests to Salesforce

Batch size: **Maximum 25 records per request**

---

## 3. External Server API

**Port:** 8083

Purpose:

* Connects to the external public API.

External API Used:
https://jsonplaceholder.typicode.com

Endpoints:

GET /api/users
GET /api/posts

Example external calls:

https://jsonplaceholder.typicode.com/users
https://jsonplaceholder.typicode.com/posts

---

## 4. Salesforce System API

**Port:** 8084

Purpose:

* Connects MuleSoft with Salesforce.
* Executes **Salesforce Composite API requests**.

Endpoint:
POST /api/composite

Operations:

* Upsert Accounts
* Upsert Contacts
* Link Contacts to Accounts using referenceId

---

# Data Transformation

The Process API merges two datasets:

Users → Company
Posts → Employees

Example structure produced:

[
{
"companyId": 1,
"companyName": "Leanne Graham",
"employees": [
{
"id": 10,
"title": "Post title"
}
]
}
]

This is then converted into **Salesforce Composite API requests**.

---

# Salesforce Integration

Salesforce objects used:

Account
Contact

Relationships:

Account → Parent
Contact → Child

Contacts are linked to accounts using:

@{refAccount.id}

Example Composite Request:

{
"method": "PATCH",
"url": "/services/data/v60.0/sobjects/Contact/External_Id__c/10",
"referenceId": "refContact10",
"body": {
"LastName": "Post Title",
"AccountId": "@{refAccount1.id}"
}
}

---

# Environment Configuration

Properties are managed using **YAML files**.

Example: `local.yaml`

http:
host: "0.0.0.0"
port: "8082"

external:
api:
host: "localhost"
port: "8083"

salesforce:
system:
host: "localhost"
port: "8084"

---

# Project Structure

Each API follows the **4-XML MuleSoft structure**:

global.xml
interface.xml
impl.xml
error.xml

Explanation:
-----
global.xml → configurations
interface.xml → API endpoints
impl.xml → business logic
error.xml → centralized error handling

