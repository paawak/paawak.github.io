---
layout: post
title: Example of creating Cucumber based BDD tests using JUnit5 and Spring Dependency Injection
author:
  display_name: paawak
  login: paawak
  email: paawak@gmail.com
  url: ''
author_login: paawak
author_email: paawak@gmail.com
date: '2021-10-12 23:00:00 +0530'
categories:
- java
- spring
- cucumber
tags:
- java
- cucumber
- tests
---
# Introduction
I am very passionate about testing. Have been so for very many years now. I am a firm believer of the idea that developers themselves should take responsibility of writing high quality tests. And the tests should not be limited to *Unit Tests*, but should also have *Integration Tests*, touching all layers of the Application. Gone are the days when testing was frowned upon by the developer community, and was the solely, exclusively only owned by the so called *Testing Team*. As we work in more Agile environment, where frequent releases is the norm, *Testing*, especially, *Integration Testing* has become an important tool to make this happen with adequate confidence. The norm is to now run our Integration Tests in our CI/CD Pipeline, in an automated way, so that we receive instant feedback.

BDD is one of the best ways to describe the behavior of an Application that is crisp and easy to understand. I use BDD in most of my Application these days along with Cucumber. In this post, I will not harp about the merits of either, but just assume people are already familiar with these terms.

What we will be discussing today is how to write elegant Integration Tests using Cucumber, JUnit5 and Spring for Dependency Injection.

## Cucumber and JUnit5
I have been using Cucumber for close to 5 years. And I have seen it evolve. The one issue I have always felt recently is the apparent lack of support for JUnit5. Of course this past year things have improved a lot, but I remember, I had to struggle to run my Cucumber tests on JUnit5 exclusively. The reason is scant documentation.

## Cucumber and Dependency Injection
Are you serious? What Dependency Injection? This is just some throwaway testing code. Why do we need a DI Framework here?

Well, people who have used Cucumber in their Java Projects, I am pretty sure, that at some point, they have faced this issue about how to share State or exchange information between Step Definitions? I have seen people go crazy and use lot of static methods and non final static variables to achieve that. Of course, that comes with a huge maintenance cost, and ultimately makes the code brittle and unreadable. And this is where a good DI Framework comes into rescue. Cucumber supports a host of [DI Frameworks](https://cucumber.io/docs/cucumber/state/#dependency-injection). Today, however, we will limit ourselves to using Spring.

# What are we testing?
Well, lets pretend that we have a library service, where we have created just 2 broad REST endpoints related to Authors and Books. The Service is written in Spring Boot, with an in-memory H2 Database. The REST end points that are exposed, allow us to list these items and then search them with a partial name. These are GET calls. We also have a single PUT call that allows us the change the Author of a Book.

We would be writing integration tests that will make REST calls to these endpoints and make sure that they are returning expected results.

# Lets build
## pom.xml
From Cucumber 7.x release, they have introduced a BOM. This has made our lives much easier. First, we will define these BOMs under the *dependencyManagement* section as shown below:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>io.cucumber</groupId>
      <artifactId>cucumber-bom</artifactId>
      <version>7.0.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
    <dependency>
      <groupId>org.junit</groupId>
      <artifactId>junit-bom</artifactId>
      <version>5.8.1</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Lets then move on to add the actual dependencies:

```xml
<dependency>
  <groupId>io.cucumber</groupId>
  <artifactId>cucumber-java</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>io.cucumber</groupId>
  <artifactId>cucumber-spring</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>io.cucumber</groupId>
  <artifactId>cucumber-junit-platform-engine</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.junit.platform</groupId>
  <artifactId>junit-platform-suite</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-api</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.junit.platform</groupId>
  <artifactId>junit-platform-console</artifactId>
  <scope>test</scope>
</dependency>
```

## The TestRunner
This is the key class that helps JUnit5 detect and run Cucumber feature files. There is a subtle difference between 6.x and 7.x. In 6.x, we used to have the *@Cucumber* annotation. In 7.x, they have deprecated this and recommend using the *JUnit Platform Suite* as shown below.

```java
import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.SelectClasspathResource;
import org.junit.platform.suite.api.Suite;

import io.cucumber.core.options.Constants;

@Suite
@SelectClasspathResource("cukes")
@ConfigurationParameter(key = Constants.PLUGIN_PUBLISH_QUIET_PROPERTY_NAME, value = "true")
@ConfigurationParameter(key = Constants.FILTER_TAGS_PROPERTY_NAME, value = "@book or @author")
public class RunCucumberTest {

}
```
By convention, the feature files are expected to be under the same package as the TestRunner. In case it is different, we can specify that by using the *@SelectClasspathResource("cukes")* annotation.

# Sources
The source code can be found here: <https://github.com/paawak/spring-boot-demo/tree/master/cucumber/cucumber-with-junit5-spring>.
