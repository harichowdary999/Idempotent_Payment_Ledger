# 💳 Idempotent Payment Ledger & Double-Spend Prevention Engine

A high-concurrency payment processing system built with **Java Spring
Boot, PostgreSQL, Python, and Data Structures & Algorithms** to prevent
duplicate transactions, ensure financial consistency, and verify ledger
integrity.

## 📌 Project Overview

In digital payment systems such as UPI and online payment gateways,
network failures and client-side retries can cause the same payment
request to be submitted multiple times.

Without proper concurrency control and idempotency mechanisms, these
duplicate requests can result in:

-   Duplicate payment deductions.
-   Inconsistent account balances.
-   Corrupted transaction records.
-   Financial reconciliation problems.

This project addresses these challenges through **idempotent payment
processing, database-level concurrency control, double-entry ledger
principles, and cryptographic integrity verification**.

## 🎯 Key Features

-   **Idempotent Payment Processing:** Prevents duplicate transactions
    using unique `Idempotency-Key` headers.
-   **Double-Spend Prevention:** Uses PostgreSQL transactions and
    pessimistic row-level locking to protect account balances.
-   **ACID Transactions:** Maintains database consistency during
    concurrent payment requests.
-   **Bloom Filter:** Uses an in-memory probabilistic data structure to
    quickly filter previously observed transaction keys.
-   **Merkle Tree Auditing:** Uses SHA-256 hashing to verify the
    integrity of transaction records.
-   **Chaos Load Testing:** Simulates hundreds of concurrent duplicate
    payment requests using Python.
-   **Live Audit Dashboard:** Displays transaction activity, ledger
    integrity, and system audit results.
-   **Automated CI/CD:** Uses GitHub Actions to build, test, audit, and
    deploy the project.

## 🏗️ System Architecture

``` mermaid
flowchart TD
    A[Client / Payment Request] --> B[Java Spring Boot API]
    B --> C[Idempotency Key Validation]
    C --> D[Bloom Filter]
    D --> E[PostgreSQL Database]
    E --> F[Transaction Processing]
    F --> G[Row-Level Locking]
    G --> H[Payment Ledger]
    H --> I[Python Merkle Auditor]
    J[Python Chaos Tester] --> B
    I --> K[Audit Report JSON]
    K --> L[HTML / CSS / JS Dashboard]
    M[GitHub Actions] --> B
    M --> J
    M --> I
    M --> L
```

### System Components

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  Java Spring Boot                    Payment REST API and business logic

  PostgreSQL                          Account balances, idempotency
                                      records, and ledger storage

  Bloom Filter                        Fast probabilistic transaction-key
                                      lookup

  Merkle Tree                         Cryptographic ledger integrity
                                      verification

  Python                              Concurrent load testing and ledger
                                      auditing

  HTML, CSS, JavaScript               Interactive payment audit dashboard

  GitHub Actions                      Automated build, testing, and
                                      deployment
  -----------------------------------------------------------------------

## 🛠️ Technology Stack

  Category          Technologies
  ----------------- ----------------------------
  Backend           Java 17, Spring Boot
  Database          PostgreSQL 15
  Database Access   Spring Data JPA, Hibernate
  API Testing       Postman
  Data Structures   Bloom Filter, Merkle Tree
  Cryptography      SHA-256
  Automation        Python 3.11
  Load Testing      Python ThreadPoolExecutor
  Frontend          HTML5, CSS3, JavaScript
  CI/CD             GitHub Actions
  Deployment        GitHub Pages

## ⚙️ How the System Works

### 1. Payment Request

The client sends a payment request with a unique idempotency key.

Example:

``` http
POST /api/payments
Content-Type: application/json
Idempotency-Key: TXN-999
```

Request body:

``` json
{
  "senderId": "ACC-101",
  "receiverId": "ACC-202",
  "amount": 500.00
}
```

### 2. Idempotency Validation

The backend checks the incoming idempotency key.

-   The Bloom Filter performs a fast probabilistic lookup.
-   A possible existing key is verified against PostgreSQL.
-   A previously processed key returns its stored response.
-   A new key proceeds to payment processing.

The database remains the source of truth for idempotency.

### 3. Concurrent Transaction Protection

PostgreSQL transaction management and pessimistic row-level locking
protect account balances during simultaneous payment requests.

The payment operation ensures:

-   Sufficient account balance.
-   Valid sender and receiver accounts.
-   Atomic balance updates.
-   Consistent ledger records.
-   Prevention of duplicate processing.

### 4. Ledger Integrity Verification

The Python auditing module retrieves transaction records from
PostgreSQL.

It then:

1.  Generates SHA-256 hashes for ledger records.
2.  Combines hashes in pairs.
3.  Repeats the hashing process until one root hash remains.
4.  Generates the Merkle Root.
5.  Exports the audit results as JSON.

Any modification to an included transaction record changes the resulting
Merkle Root.

## 🧠 Data Structures & Algorithms

### A. Bloom Filter

A Bloom Filter is a space-efficient probabilistic data structure used to
test whether an element may exist in a set.

**Purpose:** Quickly filter previously observed idempotency keys before
performing database lookups.

**Implementation:** - Bit array for storing hash positions. - Three
independent hash functions. - Fast membership checks. - False positives
are possible, but false negatives are not expected when correctly
maintained.

**Time Complexity:** O(k) per lookup, where k is the number of hash
functions.

**Space Complexity:** O(m), where m is the bit-array size.

### B. Merkle Tree

A Merkle Tree is a hash-based tree structure used to verify the
integrity of stored transaction records.

**Purpose:** Detect unauthorized modifications to payment ledger
records.

**Implementation:** - SHA-256 transaction hashing. - Pairwise hash
combination. - Bottom-up tree construction. - Merkle Root generation. -
Transaction integrity verification.

**Time Complexity:** - Tree construction: O(n) - Individual inclusion
proof verification: O(log n)

**Space Complexity:** O(n) for storing the complete tree.

## 🗄️ Database Design

The project uses PostgreSQL to maintain account information, idempotency
records, and payment ledger entries.

### Database Tables

**1. accounts**

Stores customer account information and available balances.

  Column       Data Type       Description
  ------------ --------------- ---------------------
  account_id   VARCHAR(36)     Primary key
  owner_name   VARCHAR(100)    Account holder name
  balance      DECIMAL(15,2)   Available balance
  updated_at   TIMESTAMP       Last updated time

**2. idempotency_keys**

Stores unique request keys and their corresponding responses.

  Column            Data Type     Description
  ----------------- ------------- -----------------------
  idempotency_key   VARCHAR(64)   Primary key
  request_hash      VARCHAR(64)   Request payload hash
  response_status   INT           HTTP response status
  response_body     TEXT          Stored response
  created_at        TIMESTAMP     Request creation time

**3. ledger_entries**

Stores payment transaction records.

  Column            Data Type       Description
  ----------------- --------------- ------------------------
  entry_id          SERIAL          Primary key
  idempotency_key   VARCHAR(64)     Unique transaction key
  sender_id         VARCHAR(36)     Sender account
  receiver_id       VARCHAR(36)     Receiver account
  amount            DECIMAL(15,2)   Transaction amount
  row_hash          VARCHAR(64)     SHA-256 record hash
  created_at        TIMESTAMP       Transaction time

### Database Integrity Constraints

-   Primary key constraints.
-   Foreign key constraints.
-   Unique idempotency keys.
-   Positive transaction amount validation.
-   Non-negative account balance validation.
-   Atomic payment transactions.

## 🧪 Chaos Testing & Double-Spend Verification

The Python chaos testing module simulates concurrent payment requests to
test the system under high contention.

### Test Scenario

  Parameter                Value
  ------------------------ ---------
  Sender Account           ACC-101
  Receiver Account         ACC-202
  Initial Sender Balance   ₹10,000
  Transaction Amount       ₹500
  Concurrent Requests      500
  Idempotency Key          TXN-999

### Expected Result

Only one payment should be processed successfully, even when all 500
requests use the same idempotency key.

  Metric                     Expected Value
  ------------------------ ----------------
  Initial Sender Balance            ₹10,000
  Successful Payments                     1
  Duplicate Requests                    499
  Total Amount Deducted                ₹500
  Final Sender Balance               ₹9,500
  Final Receiver Balance               ₹500

This test demonstrates the effectiveness of idempotency and database
concurrency control.

## 📊 Payment Audit Dashboard

The frontend dashboard provides a visual representation of payment
processing and ledger integrity.

### Dashboard Components

-   Total transaction count.
-   Successful and rejected payments.
-   Duplicate request statistics.
-   Account balance information.
-   Transaction audit trail.
-   Merkle Root display.
-   Ledger integrity verification status.
-   Lock contention statistics.

The dashboard uses generated audit data from:

``` text
frontend/audit-report.json
```

## 🔄 CI/CD Pipeline

GitHub Actions automates the project's build, testing, and audit
workflow.

### Pipeline Workflow

1.  Checkout the GitHub repository.
2.  Set up Java 17.
3.  Set up Python 3.11.
4.  Start PostgreSQL using GitHub Actions services.
5.  Initialize the database schema.
6.  Build the Spring Boot application using Maven.
7.  Start the payment API.
8.  Execute the Python concurrent chaos test.
9.  Run the Merkle Tree integrity auditor.
10. Generate the audit report JSON.
11. Deploy the audit dashboard using GitHub Pages.

### CI/CD Configuration

``` text
.github/
└── workflows/
    └── ci-cd.yml
```

## 📁 Repository Structure

``` text
idempotent-payment-ledger/
│
├── backend-java/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com/payment/ledger/
│   │   │   │       ├── controller/
│   │   │   │       ├── service/
│   │   │   │       ├── repository/
│   │   │   │       ├── entity/
│   │   │   │       ├── config/
│   │   │   │       └── dsa/
│   │   │   │           └── BloomFilter.java
│   │   │   └── resources/
│   │   │       └── application.properties
│   │   └── test/
│   └── pom.xml
│
├── auditor-python/
│   ├── chaos_test.py
│   ├── merkle_auditor.py
│   └── requirements.txt
│
├── frontend/
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   └── audit-report.json
│
├── sql/
│   └── schema.sql
│
├── .github/
│   └── workflows/
│       └── ci-cd.yml
│
├── .gitignore
└── README.md
```

## 🚀 Getting Started

### Prerequisites

Install the following tools:

-   Java Development Kit (JDK) 17
-   Apache Maven
-   PostgreSQL 15 or later
-   Python 3.11
-   Git
-   Postman

### Clone the Repository

``` bash
git clone https://github.com/YOUR_USERNAME/idempotent-payment-ledger.git
```

``` bash
cd idempotent-payment-ledger
```

### Project Modules

The system is organized into four major modules:

1.  **Payment Backend:** Handles REST API requests, idempotency, and
    payment processing.
2.  **Database Layer:** Manages accounts, transaction records, and
    database consistency.
3.  **Python Auditor:** Performs concurrent payment testing and Merkle
    Tree verification.
4.  **Audit Dashboard:** Displays payment activity and ledger integrity
    reports.

## 🔐 Core Engineering Concepts

This project demonstrates practical applications of:

-   Idempotency in distributed systems.
-   Race conditions and concurrency control.
-   Database transactions and ACID properties.
-   Pessimistic locking.
-   Double-spend prevention.
-   Double-entry bookkeeping principles.
-   Probabilistic data structures.
-   Cryptographic hashing.
-   Merkle Trees and data integrity.
-   Concurrent programming.
-   Automated testing and CI/CD.

## 🎓 Project Objective

The primary objective is to design and demonstrate a reliable payment
processing engine capable of handling duplicate requests and concurrent
transactions without compromising financial consistency.

The project combines **Backend Development, Database Engineering, Data
Structures & Algorithms, Python Automation, and DevOps practices** in
one practical system.

------------------------------------------------------------------------

**Built with Java, PostgreSQL, Python, and Data Structures &
Algorithms.**
