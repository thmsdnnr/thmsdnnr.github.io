---
layout: post
title:  "Introduction to Databases"
date:   2018-3-14 06:56:04 -0500
description: A walk through the fundamentals of databases. What are they, how do they differ from file systems, and what are the advantages to using them, in terms of the specific data problems they solve? Also, a short overview of SQL vs NonSQL databases, and the pros and cons of each for a given application.
author: Thomas Danner
lang: en_US
categories: compsci
tags: JS, computer science, databases, SQL, NoSQL, B-tree, queries, data modeling, data integrity, CRUD, ACID, transactions
comments: true
---

## What is a database?

A database is a hub for piping information to and from your app. A database performs many functions. Some of the most common operations are CRUD: [create, read, update, and delete](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete).

Imagine that you're creating an app to connect cat lovers. An enthusiastic user visits the site and enters her information, eager to join. When she clicks "Join", the app does a couple of things.

1. READ from the database to make sure the cat enthusiast's username (`caturday114`) is not already taken. In this case, we're good.
2. CREATE a new record in the database with the user's username, password ([hashed](https://en.wikipedia.org/wiki/Cryptographic_hash_function) so as not to be stored in plaintext), and other information.

Our eager user has logged into the app. Hooray! She quickly clicks "Upload" to upload pictures of her kitten sleeping on the sofa. When she does, the app CREATEs a new record containing the image and meta information (title, caption, date and time uploaded).

The photo now appears, but whoops, `caturday114` realized that she mistyped her cat's name in the photo. She clicks "Edit" and types in the correct name. The app tells the database to UPDATE the existing record for the photo with the correct spelling.

One day far in the future, `caturday114` realizes that she no longer likes the social media platform for cats. She goes into "Edit Profile" and chooses "Delete Account". The app sends a DELETE message to the database, and her record is deleted (or, in some cases, a delete flag is set, and the record is kept, in case she changes her mind later -- which could be fixed with an...UPDATE!).

There's nothing magical happening here. You might ask yourself, hmm, why do I even need a database? Why can't I just create a new .txt file on the server for each user that joins and stash all of their information in there?

Great question.

## Isn't a db just a file system?

Databases are stored *on* a file system. But they are also different from a file system.

Some key advantages of a database over the file system include:

### Faster Retrieval

It's much easier to ask a database "How many cat fans have been here for over one year" than it is to look in a folder named `users`, open every single .txt file, parse the contents for JOIN_DATE, and build up an array of users with JOIN_DATE >= Now - 1 Year Ago. Asking the database questions is known as querying.

### Graceful Handling of Simultaneous Requests

Imagine you are checking your cat text files for JOIN_DATE, but now another user wants to see a list of all the users who they share a birthday with, so that they can plan a big cat celebration. What happens when your first request already has a file open? If you're just both reading at the same time, it's fine. But imagine you're both writing at the same time, and imagine that you might both be writing to the same field. Everyone's had this experience with Google Docs before. Chaos ensues.

### Optimized Storage

Since databases know what type of data they will store, they can optimize the arrangement of this information on the disk to minimize the number of times they will have to spin up a hard drive to gather information. If the database can deliver the result directly from memory, the query will execute faster, and fewer resources will be required to serve it.

One key concept in database querying is indexing. We've already talked about [Binary Search Trees](http://thmsdnnr.com/tutorials/compsci/2018/02/01/binary-trees.html) and [heaps](http://thmsdnnr.com/tutorials/compsci/2018/02/12/binary-heaps.html). Enter the [B-tree](https://en.wikipedia.org/wiki/B-tree#B-tree_usage_in_databases). B-trees maintain an index of a given piece of information in the database that eases searching for records corresponding to that information. The strategy is to make the information accessible in O(log N) time with divide and conquer.

### Distributed Storage

It seems hard to imagine how we could run out of space to store text files containing user information. One billion users at 1 kB each, and we're looking at a terabyte. But imagine that each user also has on average 3 or 4 cat pictures at 1mb each. Suddenly, we're in the Petabyte range: 1,000 terabytes of data. Too big to store on any one hard drive.

Even if we could, what happens when one of those hard drives goes down? Databases support horizontal scaling out of the box through clustering.

## Transactions

Databases have rules for handling requests. Transactions are a key concept for maintaining [data integrity](https://en.wikipedia.org/wiki/Data_integrity). What's data with bad integrity? Data that does not reflect the actual state of the application due to a fault that occurred during storage.

The canonical example is the bank account funds transfer. Jim is sending money to Karen for her catsitting services. Jim requests to send $30. The pseudocode is something like this:

```
1. Find Jim in Database. Check his account balance.
2. Find Karen in Database. Load her account.
3. Check Jim's account balance.
4. If Jim's account balance is >=$30, reduce his balance by $30.
X_X sysadmin trips over CAT-5 cord, plucking it from ethernet jack
5. Add that $30 to Karen's account balance.
```

Karen taps her foot anxiously. Where's the $30? Her own cat needs some Fancy Feast. She texts Jim. Jim checks his bank account. "I already sent it, Karen. It's showing a withdrawal." Karen: "Jim, that's impossible! I don't have the money!"

The bank should have implemented a database system that uses transactions. It sure would have prevented this fiasco. Transactions have four key concepts Atomicity, Consistency, Isolation, and Durability [(ACID)](https://en.wikipedia.org/wiki/ACID).

### Atomicity

It's all or nothing. Wrap the above pseudocode into a transaction. Either all 5 steps happen and we [commit](https://en.wikipedia.org/wiki/Commit_(data_management)) the changes, or none of them happen and we [rollback](https://en.wikipedia.org/wiki/Rollback_(data_management)) to a previous state, where Jim's account balance does not have $30 withdrawn.

### Consistency

Consistency requires data to meet certain constraints. In the case of a transfer, the constraints might include:

* WITHDRAWAL + DEPOSIT = 0 (which was not met)
* WITHDRAWAL AMT <= ACCOUNT BALANCE
* WITHDRAWAL CURRENCY = DEPOSIT CURRENCY

### Isolation

Isolation requires that two transactions cannot interact with each other while in progress. Say that Jim's fund transfer is still in progress to Karen, and at the same time, Karen is buying some Fancy Feast online for $10, but she only has $5 in her account.

If the transactions are isolated, Karen's account will be overdrawn -$5 (even though she has that $30 coming!). Then the $30 will be added, and she'll be back to $25.

Maintaining isolation avoids the situation where one transaction acts on data from an incomplete transaction, and then the incomplete transaction fails. For instance, in the case above, Jim's transfer fails to go through. We can't assume that Karen has $35 before Jim's transaction finished. We don't know if someone will trip over the cable.

If we didn't have isolation, Jim's transaction fails and is properly rolled back. Jim's balance is unchanged. But since we knew the transaction was in progress, we credited Karen's account. We created $30 that didn't exist before. Unless this bank is the Federal Reserve, this is a bug!

### Durability

Durability guarantees that completed transactions will persist. Persistence means that the data is written to some durable medium in a reliable way, like a hard drive (typically multiple hard drives). If we didn't have persistence, we could have the situation where the payment goes through, Karen sees her $30, Jim sees his -$30, they both go to bed and wake up in the morning, and Jim's been refunded.

## SQL vs NoSQL

What's SQL? Structured Query Language. Relational databases. What's NoSQL? Non-SQL. Non-relational databases.

What's relational? Related data is held in tables. Imagine that we have an Excel spreadsheet for each slice of data we're interested in for the cat app. There'd be a Users spreadsheet, a Cats spreadsheet, a Photos spreadsheet, and so on.

### Relational Databases

In a few sentences. Tables are made of rows. Each row contains data elements designated by columns. Our Users table would have columns firstName, lastName, username, password, etc. Good table design mandates that as many values as possible are filled out for each row (minimize nulls). Data is stored in a [normalized fashion](https://en.wikipedia.org/wiki/Database_normalization). This means:
* No duplicated column data (e.g., storing cell # and home # in the same column)
* Rows have an identifier known as a Primary Key that uniquely identifies a row.
* Tables do not get too "wide" -- groups of related information are broken out into individual tables and linked together.
* Tables are linked through their Primary Key. A Foreign Key is a unique identifier used to link tables. Rows in Table A have primary keys that match with Table B's.

One advantage of SQL is structured queries -- the language of SQL is standardized (at least as well as your average web standard) and widely known. If you can write a MySQL query, you can write an MSSQL Server Query.

### Non-Relational Databases

NoSQL databases are handy for modeling data that does not have a clear tabular form. Data is stored in key-value pairs, graphs, or documents (that are often XML or JSON). Some advantages of NoSQL databases are:

1. NoSQL databases are usually quicker than relational databases.
2. NoSQL does not require data to be rigidly structured. Data shape can change over time.
3. NoSQL scales more easily across machines since large tables do not have to be shared.

A potential disadvantage of NoSQL is the lack of standardization in query languages. This can make it a bit trickier to get up to speed with a new system or to port legacy code.

## TIL

* Create, read, update, delete, the four common database operations
* Atomicity, consistency, isolation, durability: data integrity through transactions
* Query Language: a means to ask the database questions about the data it contains
* Indexing: speeding up data retrieval by storing values in a readily-traversed structure
* Relational vs Non-Relational Databases: tabular vs non-tabular storage

Databases might seem really complex and scary. In fact, they are simple and useful. Have a look at these [different database model types](https://en.wikipedia.org/wiki/Database_model) to familiarize yourself with the concepts.

We intentionally left out the specifics of query syntax and database vendors in this tutorial. If you understand the basic concepts behind databases and, most important, the why behind them, you'll see which solutions make sense to solve your problem. From there, with a well-founded approach, basic data modeling and coding simple queries is (almost) trivial.
