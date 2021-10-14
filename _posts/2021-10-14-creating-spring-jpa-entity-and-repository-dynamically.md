---
layout: post
title: Creating Spring JPA Entity And Repository Dynamically
author:
  display_name: paawak
  login: paawak
  email: paawak@gmail.com
  url: ''
author_login: paawak
author_email: paawak@gmail.com
date: '2021-10-14 15:00:00 +0530'
categories:
- java
- spring
- jpa
tags:
- java
- jpa
- spring
---
# Introduction
At the outset, be warned that this is not for the fainthearted. Be prepared for a long and arduous read. Having said that, lets begin.

Many a times, I have wondered, if I can ever create a JPA Entity and a JPA Repository class dynamically, on the fly? Well, its certainly possible, though a bit complicated.

# Library REST Service
We will be building a mock library service with Spring Boot and a in-memory H2 Database. There are broadly 2 REST endpoints:
1.  Author
1.  Book

Both of these allow us to fetch all items, and also search by partial name. These are GET calls.

On the Book, we have an additional PUT call that allows us the change the Author of a Book.

# Start Simple
We will build a very normal JPA based Spring Boot Application. We have called this the *static-jpa-repo-simple*. It consists of 2 Entities and JPA Repos for Book and Author respectively.

This is the Book entity:

```java
@Entity
@Table(name = "book")
@Data
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column
    private String name;

    @OneToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "author_id")
    private Author author;

}
```

And this is the corresponding JPA Repository:

```java
public interface BookDao extends CrudRepository<Book, Integer> {

    List<Book> findByNameContainingIgnoreCase(String name);

    @Transactional
    @Modifying
    @Query("update Book set author.id = :authorId where id = :bookId")
    int updateAuthor(int bookId, int authorId);

}
```

The first method *findByNameContainingIgnoreCase()*, as the name indicates, would do a case-insensitive search on the *name* of a *Book*.

The second method *updateAuthor()* would update an *Author* in a given *Book*.

Pretty mundane, I would say. If you browse to the Swagger Page: <http://localhost:8080/swagger-ui/>, this is how it looks like:

![Swagger Page for Dynamic JPA](../assets/2021/10/dynamic-jpa-swagger.png)

## Sources
These sources can be found here: <https://github.com/paawak/spring-boot-demo/tree/master/dynamic-jpa/static-jpa-repo-simple>

# Test Harness
At this point, before we delve any further, we will write a BDD/Cucumber based test harness to test all our REST endpoints. Run the main class such that the Application starts at *http://localhost:8080*. Then, run the *RunCucumberTest*. This is very similar to our previous entry: <https://palashray.com/example-of-creating-cucumber-based-bdd-tests-using-junit5-and-spring-dependency-injection/>. As shown below, ensure all the tests are green.

![Run Cucumber Tests from Eclipse](../assets/2021/10/dynamic-jpa-cucumber-tests.png)

# Making JPA Dynamic
OK, so what exactly are we trying to do? We are trying to generate the JPA Entity *Book* and the JPA Repository *BookDao* dynamically using *ByteBuddy*. Again, very similar to our previous blog post: <https://palashray.com/creating-class-dynamically-using-bytebuddy-and-springboot/>.

## Abstraction
To make this happen, we have to delete both of these classes as the first step. Since many other classes like the Rest Controller depends on these 2 classes, we have to first create an abstraction for these 2.

We will start by copying the *static-jpa-repo-simple*, and renaming it to *static-jpa-repo-abstraction*. We will refactor the code to suit our needs.

### Abstracting the Entity
For the Entity, we will first create an interface, closely resembling the *Book*:

```java
public interface BookTemplate {

    int getId();

    void setId(int id);

    String getName();

    void setName(String name);

    Author getAuthor();

    void setAuthor(Author author);

}
```

We will use this interface in place of *Book*, like in our Rest layer, the *BookController*. In the next step, to further simplify our Entity, we will create a *MappedSuperClass* with all the JPA annotated fields.

```java
@MappedSuperclass
@Data
public class BookTemplateImpl implements BookTemplate {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column
    private String name;

    @OneToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "author_id")
    private Author author;

}
```

Now, our Entity has become very simple, it just needs to extend the *BookTemplateImpl* and have a few JPA annotations of its own.

```java
@Entity
@Table(name = "book")
public class Book extends BookTemplateImpl {

}
```

### Abstracting the Dao
We will look at the *BookDao* and create a simple interface that does not extend the *CrudRepository*, with only the methods that would be used by us.

```java
public interface BookDaoTemplate {

    List<BookTemplate> findByNameContainingIgnoreCase(String name);

    int updateAuthor(int bookId, int authorId);

    Iterable<? extends BookTemplate> findAll();

}
```

We would be using this in our entire project instead of the *BookDao*. As a side effect, our *BookDao* has become simpler:

```java
public interface BookDao extends BookDaoTemplate, CrudRepository<Book, Integer> {

    @Override
    @Transactional
    @Modifying
    @Query("update Book set author.id = :authorId where id = :bookId")
    int updateAuthor(int bookId, int authorId);

}
```

### Registering the JPA Repository Dynamically
We have pretty much figured out how to generate the Entity and the Repository dynamically. However, once we have generated these classes, how to dynamically register the Repository as a Spring Bean?

The answer lies in configuring a *JpaRepositoryFactoryBean*. When using Spring Boot, if we do not specify a *JpaRepositoryFactoryBean* explicitly, the *JpaRepositoriesAutoConfiguration* kicks in and starts looking for JPA Repositories in the classpath. However, we can use the *JpaRepositoryFactoryBean* manually and register our JPA Repositories ourselves, as shown below:

```java
@Configuration
public class JpaRepoConfig {

    @Bean
    public JpaRepositoryFactoryBean<AuthorDao, Author, Integer> authorRepository() {
	     return new JpaRepositoryFactoryBean<>(AuthorDao.class);
    }

    @Bean
    public JpaRepositoryFactoryBean<BookDao, Book, Integer> bookRepository() {
	     return new JpaRepositoryFactoryBean<>(BookDao.class);
    }

}
```

### Sources
The sources for the *static-jpa-repo-abstraction* can be found here: <https://github.com/paawak/spring-boot-demo/tree/master/dynamic-jpa/static-jpa-repo-abstraction>.

## Take the plunge
