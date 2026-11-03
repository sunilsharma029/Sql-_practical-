# MySQL Practical: Bank Transaction Management System

## Project objective

Is practical mein hum bank transfer system banayenge. Iske through MySQL
ke transaction commands aur unke important use-cases samjhenge:

-   `START TRANSACTION`
-   `COMMIT`
-   `ROLLBACK`
-   `SAVEPOINT`
-   `ROLLBACK TO SAVEPOINT`
-   `RELEASE SAVEPOINT`

**Database:** MySQL\
**Storage engine:** InnoDB

> Important: Transaction rollback tabhi expected tarike se kaam karega
> jab tables transactional storage engine, jaise InnoDB, use kar rahi
> hon.

------------------------------------------------------------------------

# Part 1: Database aur tables create karna

## Step 1: Database create karein

``` sql
CREATE DATABASE bank_transaction_db;
USE bank_transaction_db;
```

## Step 2: Accounts table create karein

``` sql
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_holder VARCHAR(50) NOT NULL,
    balance DECIMAL(10,2) NOT NULL,
    CHECK (balance >= 0)
) ENGINE = InnoDB;
```

## Step 3: Transactions history table create karein

``` sql
CREATE TABLE transaction_history (
    transaction_id INT PRIMARY KEY AUTO_INCREMENT,
    from_account INT NOT NULL,
    to_account INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL,
    FOREIGN KEY (from_account) REFERENCES accounts(account_id),
    FOREIGN KEY (to_account) REFERENCES accounts(account_id)
) ENGINE = InnoDB;
```

## Step 4: Sample data insert karein

``` sql
INSERT INTO accounts (account_id, account_holder, balance)
VALUES
(101, 'Rahul', 10000),
(102, 'Priya', 5000),
(103, 'Amit', 8000);
```

Check records:

``` sql
SELECT * FROM accounts;
```

### Initial account balances

    Account ID Account Holder     Balance
  ------------ ---------------- ---------
           101 Rahul              ₹10,000
           102 Priya               ₹5,000
           103 Amit                ₹8,000

------------------------------------------------------------------------

# Part 2: START TRANSACTION + COMMIT

## Use-case 1: Successful bank transfer

Rahul, Priya ko ₹2,000 transfer karta hai.

Is process mein: 1. Rahul ke account se ₹2,000 deduct honge. 2. Priya ke
account mein ₹2,000 add honge. 3. Transaction history mein record insert
hoga. 4. Teeno operations successful hone par changes commit honge.

``` sql
START TRANSACTION;

-- Step 1: Rahul se amount deduct karo
UPDATE accounts
SET balance = balance - 2000
WHERE account_id = 101;

-- Step 2: Priya ke account mein amount add karo
UPDATE accounts
SET balance = balance + 2000
WHERE account_id = 102;

-- Step 3: Transaction history maintain karo
INSERT INTO transaction_history
(from_account, to_account, amount, status)
VALUES
(101, 102, 2000, 'SUCCESS');

-- Step 4: Changes save karo
COMMIT;
```

Check accounts:

``` sql
SELECT * FROM accounts;
```

Check history:

``` sql
SELECT * FROM transaction_history;
```

### Expected balances

  Account Holder      Before    After
  ---------------- --------- --------
  Rahul              ₹10,000   ₹8,000
  Priya               ₹5,000   ₹7,000
  Amit                ₹8,000   ₹8,000

**Explanation:**

-   `START TRANSACTION` transaction start karta hai.
-   `UPDATE` statements balances change karti hain.
-   `INSERT` history record create karta hai.
-   `COMMIT` saare transaction changes save karta hai.

------------------------------------------------------------------------

# Part 3: START TRANSACTION + ROLLBACK

## Use-case 2: Transaction cancel karna

Rahul, Amit ko ₹1,000 transfer karna chahta hai, lekin transfer process
ke beech business rule ke karan transaction cancel kar di jaati hai.

``` sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 101;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 103;

INSERT INTO transaction_history
(from_account, to_account, amount, status)
VALUES
(101, 103, 1000, 'CANCELLED');

-- Saare uncommitted changes undo karo
ROLLBACK;
```

Check:

``` sql
SELECT * FROM accounts;

SELECT * FROM transaction_history;
```

**Expected result:** Rahul aur Amit ke balances unchanged rahenge.
History mein bhi is transaction ka record nahi rahega, kyunki insert bhi
rollback ho gaya.

**Important:** `ROLLBACK` history table ke insert ko bhi undo karta hai,
agar insert usi transaction mein hua ho.

Agar cancelled transaction ka audit record rakhna zaroori hai, to use
rollback ke baad alag transaction mein insert karna hoga.

------------------------------------------------------------------------

# Part 4: SAVEPOINT + ROLLBACK TO SAVEPOINT

## Use-case 3: Partial rollback

Maan lijiye ek transaction mein Rahul ke account se ₹500 deduct karna
hai, Priya ke account mein ₹500 add karna hai aur Amit ke account mein
₹300 add karna hai.

Lekin Amit wala operation cancel karna hai. Rahul aur Priya wale changes
rakhne hain.

``` sql
START TRANSACTION;

-- Operation 1
UPDATE accounts
SET balance = balance - 500
WHERE account_id = 101;

-- Checkpoint 1
SAVEPOINT after_rahul_deduction;

-- Operation 2
UPDATE accounts
SET balance = balance + 500
WHERE account_id = 102;

-- Checkpoint 2
SAVEPOINT after_priya_credit;

-- Operation 3
UPDATE accounts
SET balance = balance + 300
WHERE account_id = 103;

-- Amit wala operation undo karo
ROLLBACK TO SAVEPOINT after_priya_credit;

-- Rahul aur Priya ke changes save karo
COMMIT;
```

### Expected result

  Account Holder        Change
  ---------------- -----------
  Rahul                  -₹500
  Priya                  +₹500
  Amit               No change

**Explanation:**

-   Rahul ka deduction hua.
-   `after_rahul_deduction` checkpoint bana.
-   Priya ka amount add hua.
-   `after_priya_credit` checkpoint bana.
-   Amit ka amount add hua.
-   `ROLLBACK TO SAVEPOINT after_priya_credit` ne Amit wala update undo
    kar diya.
-   Rahul aur Priya ke changes commit ho gaye.

**Note:** Is example ka purpose partial rollback samjhana hai. Real bank
transfer mein debit aur credit ko ek saath commit ya rollback karna
chahiye. Sirf debit commit karna unsafe hai.

------------------------------------------------------------------------

# Part 5: Multiple SAVEPOINTS

## Use-case 4: Multiple checkpoints ke saath transaction

Ek transaction mein hum multiple savepoints create kar sakte hain.

``` sql
START TRANSACTION;

-- First operation
UPDATE accounts
SET balance = balance - 100
WHERE account_id = 101;

SAVEPOINT sp1;

-- Second operation
UPDATE accounts
SET balance = balance + 100
WHERE account_id = 102;

SAVEPOINT sp2;

-- Third operation
UPDATE accounts
SET balance = balance - 200
WHERE account_id = 103;

SAVEPOINT sp3;

-- Fourth operation
UPDATE accounts
SET balance = balance + 200
WHERE account_id = 102;

-- sp2 tak rollback
ROLLBACK TO SAVEPOINT sp2;

COMMIT;
```

### Kya hoga?

-   `sp1` se pehle Rahul ka ₹100 deduction hua.
-   `sp1` ke baad Priya ko ₹100 add hue.
-   `sp2` ke baad Amit se ₹200 deduct hue.
-   `sp3` ke baad Priya ko ₹200 aur add hue.
-   `ROLLBACK TO SAVEPOINT sp2` ne `sp2` ke baad ke changes undo kar
    diye.
-   `sp2` tak ke changes commit ho gaye.

### Final changes

  Account Holder        Change
  ---------------- -----------
  Rahul                  -₹100
  Priya                  +₹100
  Amit               No change

**Remember:** `ROLLBACK TO SAVEPOINT sp2` se `sp2` ke baad ke changes
undo hote hain. Savepoint `sp2` khud generally available rehta hai, jab
tak release ya transaction end na ho.

------------------------------------------------------------------------

# Part 6: RELEASE SAVEPOINT

## Use-case 5: Savepoint ko remove karna

`RELEASE SAVEPOINT` ka use kisi savepoint ko remove karne ke liye hota
hai. Ye data changes ko undo nahi karta.

``` sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 100
WHERE account_id = 101;

SAVEPOINT sp1;

UPDATE accounts
SET balance = balance + 100
WHERE account_id = 102;

-- Savepoint remove karo
RELEASE SAVEPOINT sp1;

COMMIT;
```

**Explanation:**

-   Rahul se ₹100 deduct hue.
-   `sp1` create hua.
-   Priya ko ₹100 add hue.
-   `RELEASE SAVEPOINT sp1` ne checkpoint remove kar diya.
-   `COMMIT` ne dono changes save kar diye.

**Important:** Savepoint release karne se usse pehle ya baad ke SQL
changes automatically undo nahi hote.

------------------------------------------------------------------------

# Part 7: Insufficient balance ka use-case

## Use-case 6: Validation fail hone par rollback

Rahul ke account mein ₹8,000 se kam amount hone par transfer nahi hona
chahiye.

Transaction start karne ke baad balance check karein. Agar balance
sufficient nahi hai, to transaction rollback karein.

``` sql
START TRANSACTION;

-- Current balance check
SELECT balance
FROM accounts
WHERE account_id = 101;

-- Assume application ne balance verify kiya.
-- Agar balance sufficient hai, tabhi transfer statements execute karein.

UPDATE accounts
SET balance = balance - 9000
WHERE account_id = 101;

UPDATE accounts
SET balance = balance + 9000
WHERE account_id = 102;

COMMIT;
```

**Important learning:** Upar diya gaya SQL sirf tab safe hai jab
application pehle sufficient balance verify kare aur concurrent
transactions ko bhi dhyan mein rakhe. Sirf `SELECT` karke baad mein
`UPDATE` karna race condition create kar sakta hai.

### Better database-side validation

InnoDB ke saath conditional update use karein:

``` sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 9000
WHERE account_id = 101
AND balance >= 9000;

SELECT ROW_COUNT() AS affected_rows;
```

Ab application ko `affected_rows` check karna hoga:

-   `1` = deduction successful; credit operation execute karke commit
    karein.
-   `0` = sufficient balance nahi tha; `ROLLBACK` karein.

Agar application logic mein deduction successful ho:

``` sql
UPDATE accounts
SET balance = balance + 9000
WHERE account_id = 102;

COMMIT;
```

Agar deduction fail ho:

``` sql
ROLLBACK;
```

**Note:** MySQL stored program ya application code mein conditional
logic se ye decision automatically handle kiya ja sakta hai. SQL client
mein statements ko ek-ek karke chalate waqt result khud check karna
hoga.

------------------------------------------------------------------------

# Part 8: Autocommit samajhna

MySQL mein default autocommit aam taur par enabled hota hai.

Check:

``` sql
SELECT @@autocommit;
```

-   `1` = Autocommit enabled.
-   `0` = Autocommit disabled.

Autocommit disable:

``` sql
SET autocommit = 0;
```

Ab statements automatically commit nahi hongi; transaction ko explicitly
commit ya rollback karna hoga.

``` sql
UPDATE accounts
SET balance = balance - 100
WHERE account_id = 101;

COMMIT;
```

Autocommit wapas enable:

``` sql
SET autocommit = 1;
```

**Important:** `START TRANSACTION` explicitly transaction start karta
hai, chahe autocommit enabled ho. Transaction ke `COMMIT` ya `ROLLBACK`
ke baad autocommit setting apni previous value par rehti hai.

------------------------------------------------------------------------

# Part 9: Important practical rules

1.  Transaction ke liye InnoDB jaise transactional storage engine ka use
    karein.
2.  `START TRANSACTION` current session mein transaction start karta
    hai.
3.  `COMMIT` current transaction ke changes save karta hai.
4.  `ROLLBACK` current transaction ke uncommitted changes undo karta
    hai.
5.  `SAVEPOINT` ek checkpoint create karta hai.
6.  `ROLLBACK TO SAVEPOINT` specified checkpoint ke baad ke changes undo
    karta hai.
7.  `RELEASE SAVEPOINT` checkpoint remove karta hai, data changes ko
    undo nahi karta.
8.  Full `ROLLBACK` ke baad current transaction ke savepoints bhi khatam
    ho jaate hain.
9.  `COMMIT` ke baad us transaction ko normal `ROLLBACK` se undo nahi
    kar sakte.
10. MySQL ke kuch DDL statements, jaise `CREATE TABLE`, `ALTER TABLE`
    aur `DROP TABLE`, implicit commit cause kar sakte hain. Inhe
    rollback-safe na samjhein.
11. Transaction ko unnecessarily open na rakhein; jaldi commit ya
    rollback karein.
12. Real bank transfer mein debit, credit aur transaction record ko ek
    hi transaction mein commit karna chahiye.

------------------------------------------------------------------------

# Part 10: Command summary

  Command                        Meaning
  ------------------------------ ------------------------------------
  `START TRANSACTION;`           Transaction start
  `COMMIT;`                      Changes save
  `ROLLBACK;`                    Current transaction undo
  `SAVEPOINT sp1;`               Checkpoint create
  `ROLLBACK TO SAVEPOINT sp1;`   Checkpoint ke baad ke changes undo
  `RELEASE SAVEPOINT sp1;`       Checkpoint remove
  `SELECT @@autocommit;`         Autocommit status check

------------------------------------------------------------------------

# Part 11: Student practice assignments

## Assignment 1: Basic transaction

1.  Database aur accounts table create karein.
2.  Teen bank accounts insert karein.
3.  `START TRANSACTION` se transaction start karein.
4.  Ek account se ₹500 deduct karein.
5.  Doosre account mein ₹500 add karein.
6.  `COMMIT` karke balances check karein.

## Assignment 2: Rollback

1.  Transaction start karein.
2.  Do accounts ke balances update karein.
3.  `ROLLBACK` execute karein.
4.  Check karein ki original balances restore hue ya nahi.

## Assignment 3: Savepoint

1.  Transaction start karein.
2.  Pehla account update karein.
3.  `sp1` create karein.
4.  Doosra account update karein.
5.  `sp2` create karein.
6.  Teesra account update karein.
7.  `ROLLBACK TO SAVEPOINT sp2` execute karein.
8.  `COMMIT` karein.
9.  Final balances verify karein.

## Assignment 4: Release savepoint

1.  Transaction start karein.
2.  Account update karein.
3.  Savepoint create karein.
4.  Doosra account update karein.
5.  Savepoint release karein.
6.  Commit karein.
7.  Verify karein ki dono updates save hue.

## Assignment 5: Bank transfer validation

1.  Transfer amount input lein.
2.  Sender ka balance check karein.
3.  Balance sufficient ho to amount deduct karein.
4.  Receiver ke account mein amount add karein.
5.  Transaction history mein record insert karein.
6.  Saare operations successful hon to commit karein.
7.  Kisi operation mein failure ho to rollback karein.

------------------------------------------------------------------------

# Quick revision

-   `START TRANSACTION` = Transaction shuru.
-   `COMMIT` = Changes save.
-   `ROLLBACK` = Uncommitted changes undo.
-   `SAVEPOINT` = Checkpoint create.
-   `ROLLBACK TO SAVEPOINT` = Partial rollback.
-   `RELEASE SAVEPOINT` = Checkpoint remove.

**Golden rule:** Bank transfer jaise critical operation mein debit aur
credit dono successful hone chahiye. Dono ko ek transaction mein rakhein
aur sirf tab `COMMIT` karein jab poora operation successful ho.
