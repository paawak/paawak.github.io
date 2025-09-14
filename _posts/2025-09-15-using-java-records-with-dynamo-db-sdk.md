---
layout: post
background: '/assets/banner/HemkutHill_12.jpg'
title: Using Java records with DynamoDB SDK
author:
  name: Palash Ray
  
  email: paawak@gmail.com
  url: 'https://www.linkedin.com/in/palash-ray/'


date: '2025-09-15 0:50:00 +0530'
categories:
- java
- aws
- dynamodb
tags:
- java
- aws
- dynamodb
---
# Duplicate row is created after SortKey updated
Recently, when working with AWS DynamoDB, I made another discovery. The Partition Key and Sort Key or any other Index Key, for that matter are always immutable.

What do I mean by that? Let us digress a little and talk about the write operations that are available. They are __PutItem__ and __UpdateItem__, analogous of __Insert__ and __Update__. However, due to the very nature of DynamoDB, both of these operations are effectively __Upsert__.

## The standard table POJO
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
## The Enhanced DynamoDB Client TableSchema

Now, we will setup the Enhanced DynamoDB Client and TableSchema as shown below:

```java
@Configuration
public class DynamoDBConfig {

    @Bean
    public DynamoDbEnhancedClient dynamoDbEnhancedClient() {
	return DynamoDbEnhancedClient.create();
    }

    @Bean
    public DynamoDbTable<Book> bookTable(DynamoDbEnhancedClient dynamoDbEnhancedClient) {
	return dynamoDbEnhancedClient.table("book", TableSchema.fromBean(Book.class));
    }

}
```

## Insert a new Book

With the __enhanced dynamo-db client__, let us perform an insert operation:

```java
    public void insertBook(DynamoDbTable<Book> bookTable) {
	Book newBook = new Book();
	newBook.setId("1111");
	newBook.setPublishedOn(LocalDateTime.of(2025, 7, 8, 13, 10));
	newBook.setTitle("AAAA");
	newBook.setDescription("BBBB");
	newBook.setAuthorId("ccc");

	bookTable.putItem(newBook);
    }
```

## Update an existing Book
Now, we will update the same book with id __111__, as below:

```java
    public void updateBookPublishedOn(DynamoDbTable<Book> bookTable, Book existingBook, LocalDateTime newDate) {
	existingBook.setPublishedOn(newDate);

	bookTable.updateItem(existingBook);
    }
```

Note that I am updating the __publishedOn__ attribute. My expectation would be to see that the Book with id __111__ has been updated with the new __publishedOn__ date. However, what happens is really unexpected. We find that there are now 2 rows with the same __bookId 111__, but different Sort Keys.

Though unexpected, this is a conscious design decision that was taken by DynamoDB, and that is how most NoSQL DBs work. Here Partition Keys and Sort keys are immutable. Once written, they cannot be modified. Again, the HashMap analogy comes handy. Can you modify the HashKey of a record inside of a HashMap? Obviously, no.

# Solution: Immutable Table Schema with java record

The solution really is to use the __@DynamoDbImmutable__ annotation to have an immutable schema using builder pattern. [The example from the official site](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/ddb-en-client-use-immut.html) uses a class, but to my pleasant surprise, we can use even __records__ along with Lombok Builder. 

```java
@DynamoDbImmutable(builder = ImmutableBook.ImmutableBookBuilder.class)
@Builder
public record ImmutableBook(@DynamoDbPartitionKey String id, @DynamoDbSortKey LocalDateTime publishedOn, String title,
	String description, String authorId) {
}

```

And the below configuration for creating a table schema:

```java
    @Bean
    public DynamoDbTable<ImmutableBook> immutableBookTable(DynamoDbEnhancedClient dynamoDbEnhancedClient) {
	return dynamoDbEnhancedClient.table("book", TableSchema.fromImmutableClass(ImmutableBook.class));
    }
```

Now, why is this significant?

The reason is: once your Table Schema is a record, its immutable and you can only make the rookie mistake of updating an existing record would be with difficulty, and would mostly be called out in peer reviews.

# Source Code
Can be found [here](https://github.com/paawak/spring-boot-demo/tree/master/dynamodb-demo).

# Reference
1. <https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html>
1. <https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/ddb-en-client-gs-tableschema.html>
1. <https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/ddb-en-client-gs-use.html>
1. <https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/ddb-en-client-use-immut.html>