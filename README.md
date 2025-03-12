# Fraud_Detection_Analytics SQL Project

**Project STRUCTURE** 

### 1. Databse Setup

**Database Creation**: Create a database named 'fraud_detection'.
**Table Creation**: Create a table named 'transactions'. The table contains columns named (step, type, amount, nameOrig, oldbalanceOrg, newbalanceOrig, nameDest, oldbalanceDest, newbalanceDest, isFraud, isFlaggedFraud)

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

### 3. Analyzing Fraudulent Activity over Time
**Question**: Using CTE, calculate the rolling sum of fraudulent transactions for each account over the last 5 steps

```sql
WITH rolling_fraud AS (select nameorig, step, SUM(isfraud)
OVER (PARTITION BY nameOrig ORDER BY step ROWS BETWEEN 4 PRECEDING AND CURRENT ROW)
AS fraud_rolling FROM transactions)
SELECT * FROM rolling_fraud WHERE fraud_rolling > 0;
```

### 4. Complex Fraud Detection Using Multiple CTEs
**Question**: Use multiple CTEs to identify accounts with suspicious activity, including large transfers, consecutive transactions without balance change, and flagged transactions

```sql
WITH large_transfers as (
SELECT nameOrig,step,amount FROM transactions WHERE type = 'TRANSFER' and amount >500000),
no_balance_change as (
SELECT nameOrig,step,oldbalanceOrg,newbalanceOrig FROM transactions where oldbalanceOrg=newbalanceOrig),
flagged_transactions as (
SELECT nameOrig,step FROM transactions where  isflaggedfraud = 1)
SELECT
    lt.nameOrig
FROM
    large_transfers lt
JOIN
    no_balance_change nbc ON lt.nameOrig = nbc.nameOrig AND lt.step = nbc.step
JOIN
    flagged_transactions ft ON lt.nameOrig = ft.nameOrig AND lt.step = ft.step;
```

### 5. Write a query that checks if the computed new_updated_Balance is the same as the actual newbalanceDest in the table. If they are equal, it returns those rows

```sql
WITH CTE AS (
SELECT amount, nameorig, oldbalancedest, newbalanceDest, (amount+oldbalancedest) as new_updated_Balance 
FROM transactions
)
SELECT * FROM CTE WHERE new_updated_Balance = newbalanceDest;
```
