---
layout: post
background: '/assets/banner/HemkutHill_12.jpg'
title: Designing tables in NoSQL Databases
author:
  name: Palash Ray
  
  email: paawak@gmail.com
  url: 'https://www.linkedin.com/in/palash-ray/'


date: '2025-08-17 20:00:00 +0530'
categories:
- java
- nosql
- dynamodb
tags:
- java
- nosql
- dynamodb
---

Working with no-sql databases can be a bit of a cultural shock. There are many things which are different from the usual ACID compliant Relational Databases. The greatest shocker is probably that: you cannot just query any attributes in a select query. You can only query Partition Keys, Sort Keys and Indexes. 

Long back, when I first started tinkering with Cassandra, and seeing me struggle with getting my queries right, my mentor suggested that I should start thinking of these no-SQL databases as a huge, distributed hash-map. When I want to retrieve any item from a hashmap, I obviously should know its key. In the context of a no-sql db, that key is the combination of the Partition Key and the Sort Key. This nugget of knowledge has really stayed with me. I find it very useful to design my no-sql db tables. I primarily think of how I would query them? What kind of questions would I ask the table, which, essentially is a huge hashmap.

So, the first and the only lesson: design your tables with the queries in mind. Before you start defining your Partition Key, Sort Key or other Indexes, start thinking about the kind of queries you will run on the table or the kind of questions you want an answer to.

This, I have found to be very relevant to no-sql DBs like Cassandra and DynamoDB.
