---
layout: post
background: '/assets/banner/HemkutHill_12.jpg'
title: Using Java records with DynamoDB SDK
author:
  name: Palash Ray
  
  email: paawak@gmail.com
  url: 'https://www.linkedin.com/in/palash-ray/'


date: '2025-08-23 20:00:00 +0530'
categories:
- java
- aws
- dynamodb
tags:
- java
- aws
- dynamodb
---
# Background
Working with no-sql databases can be a bit of a cultural shock. There are many things which are different from the usual ACID compliant Relational Databases. The greatest shocker is probably that: you cannot just query any attributes in select. You can only query Partition Keys, Sort Keys and Indexes. Long back, when I first started tinkering with Cassandra, and seeing me struggle with getting my queries right, my mentor suggested that I should start thinking of these no-SQL databases as a huge, distributed hash-map. When I want to retrieve any item from a hashmap, I obviously should know its key. In the context of a no-sql db, that key is the combination of the Partition Key and the Sort Key. This nugget of knowledge has really stayed with me. I find it very useful to design my no-sql db tables. I primarily think of how I would query them? What kind of questions would I ask the table, which, essentially is a huge hashmap.

Recently, when working with AWS DynamoDB, I made another discovery. The Partition Key and Sort Key or any other Index Key, for that matter are always immutable.

What do I mean by that? Let us digress a little and talk about the write operations that are available. They are __PutItem__ and __UpdateItem__, analogous of __Insert__ and __Update__. However, due to the very nature of DynamoDB, both of these operations are effectively __Upsert__.

Consider the below POJO that models a normal DynamoDB table:

```java
import lombok.Data;
import lombok.Getter;

import java.time.LocalDateTime;

import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbSortKey;

@DynamoDbBean
@Data
public class Book {

	@Getter(onMethod_ = { @DynamoDbPartitionKey })
	private String id;

	@Getter(onMethod_ = { @DynamoDbSortKey })
	private LocalDateTime publishedOn;

	private String title;

	private String description;

	private String authorId;
}
```

With the __enhanced dynamo-db client__, let us perform some basic operations:

```java
    public void writeBook(DynamoDbTable<Book> table){
		Book book = new Book();
		book.setId("1111");
		book.setPublishedOn(LocalDateTime.of(2025, 7, 8, 13, 10));
		book.setTitle("AAAA");
		book.setDescription("BBBB");
		book.setAuthorId("ccc");
		
        table.putItem(book);
    }
```


# Reference
1. <https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html>
1. <https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/ddb-en-client-gs-tableschema.html>
1. <https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/ddb-en-client-gs-use.html>