---
layout: post
background: '/assets/banner/HemkutHill_12.jpg'
title: Validating a Json file Against an Avro Schema
author:
  name: Palash Ray
  
  email: paawak@gmail.com
  url: 'https://www.linkedin.com/in/palash-ray/'


date: '2025-09-08 23:00:00 +0530'
categories:
- java
- avro
tags:
- java
- avro
---

[Apache Avro](https://avro.apache.org/docs/) is a data serialization framework. Its pretty popular for systems using Kafka, with a strict schema. Avro enforces a rather strict schema.

Where most people struggle is to write test cases and validate whether a given json message actually conforms to a given Avro schema. There is no in-built API in Avro to do that. It is a bit complicated. While, theoretically, it is possible to generate Java classes from an Avro schema, those classes are not POJOs, but they are a part of the Avro IDL (Interface Definition Language), and they extend the [SpecificRecordBase](https://avro.apache.org/docs/1.12.0/api/java/org/apache/avro/specific/SpecificRecordBase.html) class. It is more like the WSDL files of the good old days, where we used to generate some very cumbersome structures that conformed to the convoluted schemas of whatever we were trying to ship in the envelop.

In essence, if you have a simple json message that you want to read into a Java POJO that is a __SpecificBaseRecord__, thats not happening! Conversely, if you have a POJO that has the same structure as the Avro schema, and you are trying to write it into an Avro message with the [usual example](https://avro.apache.org/docs/1.12.0/getting-started-java/), you are in for a shock!

The below code fails spectacularly:

```java
    @Test
    void pojoWithSpecificDatumWriterThrowsException() throws IOException, URISyntaxException {
	// given
	Schema bookSchema =
		new SchemaParser().parse(AvroDemoTest.class.getResourceAsStream("/expected/book.avsc")).mainSchema();

	Book book = Instancio.create(Book.class);

	Path bookAvroFileLocation = Path.of("build", "book.avro");

	// when
	DatumWriter<Book> datumWriter = new SpecificDatumWriter<>();
	@SuppressWarnings("resource")
	DataFileWriter<Book> dataFileWriter = new DataFileWriter<>(datumWriter);
	dataFileWriter.create(bookSchema, bookAvroFileLocation.toFile());
	AppendWriteException result = assertThrows(AppendWriteException.class, () -> dataFileWriter.append(book));

	// then
	assertTrue(result.getMessage().startsWith("java.lang.ClassCastException: value Book"));
    }
```

And the error is so cryptic! Basically what its trying to tell you is that the Book class should be extending the __SpecificBaseRecord__.


So, whats the way out?

Simple: instead of the [SpecificDatumWriter](https://avro.apache.org/docs/1.12.0/api/java/org/apache/avro/specific/SpecificDatumWriter.html), use the [ReflectDatumWriter](https://avro.apache.org/docs/1.12.0/api/java/org/apache/avro/reflect/ReflectDatumWriter.html).

This is the complete example:

```java
    @Test
    void validatePojoAgainstSchema() throws IOException, URISyntaxException {
	// given
	Schema bookSchema =
		new SchemaParser().parse(AvroDemoTest.class.getResourceAsStream("/expected/book.avsc")).mainSchema();

	List<Book> booksInput = Instancio.createList(Book.class);

	Path bookAvroFileLocation = Path.of("build", "book.avro");

	// when
	DatumWriter<Book> datumWriter = new ReflectDatumWriter<>();
	DataFileWriter<Book> dataFileWriter = new DataFileWriter<>(datumWriter);
	dataFileWriter.create(bookSchema, bookAvroFileLocation.toFile());
	booksInput.forEach(book -> {
	    try {
		dataFileWriter.append(book);
	    } catch (IOException e) {
		throw new RuntimeException(e);
	    }
	});
	dataFileWriter.close();

	// then
	DatumReader<Book> datumReader = new ReflectDatumReader<>(Book.class);
	DataFileReader<Book> dataFileReader = new DataFileReader<>(bookAvroFileLocation.toFile(), datumReader);
	List<Book> booksOutput = new ArrayList<>();
	while (dataFileReader.hasNext()) {
	    booksOutput.add(dataFileReader.next());
	}
	dataFileReader.close();
	assertEquals(booksInput, booksOutput);
    }
```

These are the steps:
1. First, I make sure that [Book](https://github.com/paawak/blog/blob/master/code/avro-demo/src/main/java/com/swayam/blog/demo/avro/model/Book.java) is a POJO. Read you Json into an instance of __Book__.
2. Next, create an instance of __ReflectDatumWriter__.
3. Create an instance of __DataFileWriter__ and assign the __Schema__.
4. Now, write the instance of Book into it
5. Close the __DataFileWriter__
6. Read the avro file thus created with the help of the __ReflectDatumReader__
7. If you are able to successfully read the Book data that was just written into the avro file, you know that the Json message conforms to the Avro schema

The complete sources and a working project can be found in the [avro-demo](https://github.com/paawak/blog/tree/master/code/avro-demo).
