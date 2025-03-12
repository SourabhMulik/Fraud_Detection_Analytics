# Fraud_Detection_Analytics SQL Project

**Project STRUCTURE** 

### 1. Databse Setup

**Database Creation**: Create a database named 'fraud_detection'.
**Table Creation**: Create a table named 'transactions'. The table contains columns named (step INT, type VARCHAR(20), amount DECIMAL(15,2), nameOrig VARCHAR(20), oldbalanceOrg DECIMAL(15,2), newbalanceOrig DECIMAL(15,2), nameDest VARCHAR(20), oldbalanceDest DECIMAL(15,2), newbalanceDest DECIMAL(15,2), isFraud INT, isFlaggedFraud INT).

```sql
CREATE DATABASE fraud_detection;

CREATE TABLE transactions
(
    step INT,
    type VARCHAR(20),
    amount DECIMAL(15,2),
    nameOrig VARCHAR(20),
    oldbalanceOrg DECIMAL(15,2),
    newbalanceOrig DECIMAL(15,2),
    nameDest VARCHAR(20),
    oldbalanceDest DECIMAL(15,2),
    newbalanceDest DECIMAL(15,2),
    isFraud INT,
    isFlaggedFraud INT
);
```

### 2. Detecting Recursive Fraudulent Transactions
**Question**: Use a recursive CTE to identify potential money laundering chains where money is transferred from one account to another across multiple steps, with all transactions flagged as fraudulent

```sql
WITH RECURSIVE fraud_chain AS
(
SELECT nameOrig AS initial_account, nameDest AS next_account, step, amount, newbalanceorig
FROM transactions
WHERE isfraud = 1 AND type = 'TRANSFER'
UNION ALL
SELECT fc.initial_account, t.nameDest, t.step, t.amount, t.newbalanceorig
FROM fraud_chain fc
JOIN transactions t
ON fc.next_account = t.nameorig AND fc.step < t.step
WHERE t.isFraud = 1 AND t.type = 'TRANSFER'
)
SELECT * FROM fraud_chain;
```
