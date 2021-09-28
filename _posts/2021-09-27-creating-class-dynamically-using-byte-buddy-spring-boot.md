---
layout: post
title: Creating Class Dynamically Using ByteBuddy and SpringBoot
author:
  display_name: paawak
  login: paawak
  email: paawak@gmail.com
  url: ''
author_login: paawak
author_email: paawak@gmail.com
date: '2021-09-27 23:13:00 +0530'
categories:
- java
- spring
tags:
- java
- byte-buddy
---
# Introduction
In this post, we are going to talk about how we can generate a class dynamically. Welcome to the brave new world of powerful possibilities. Generating classes dynamically as a concept, is as old as the Java language itself. RMI, J2EE, etc. used this powerful concept to a large extent. With the advent of Annotations in JDK 1.5, this became very popular. You had libraries pre-processing annotations and generating proxies to do away with lot of boiler-plate code. Especially worth mentioning are the Aspect Oriented Programming concepts (or AOP) around Transaction management. Suffice it to say that Spring and lot of other popular libraries like Hibernate, would not be able to function as they do, without this.

There are a number of very powerful libraries that allow you to generate classes dynamically. I had used [Apache BCEL](https://commons.apache.org/proper/commons-bcel/) a [bit longish back](https://palashray.com/your-cup-of-byte-code-re-engineering-bcel-style/). However, today, we will explore a simple, yet powerful library called [ByteBuddy](https://bytebuddy.net/#/).

# Let's build...
I am building a trivial Web Application with Spring Boot. It uses versioned REST APIs. Nothing fancy, just that each of the end points starts with its version. For example, the V1 of my API would be <http://localhost:8080/v1/bank-item/>. Similarly, no prizes for guessing, that the V2 of my API would be <http://localhost:8080/v2/bank-item/>.

# The Challenge
I have already built the V1 of my REST end point. And of course, I have a RestController for that. Now, I am feeling too lazy to build the V2 of the same end point, as I know that would mean I have to write a DAO, a Service, the Model, the entire stack, before I even get to my RestController. And I love taking short cuts. So, can I extend my existing RestController for V1, and create another RestController such that it starts responding to my V2 requests? And can I do this dynamically too?

Assuming, we already have an existing RestController called __BankDetailController__, serving the V1 endpoint, basically what we would like to do is, create a class similar to below:

```java
@RestController
@RequestMapping("/v2/bank-item")
public class BankDetailControllerV2 extends BankDetailController {

    public BankDetailControllerV2(BankDetailService bankDetailService) {
	     super(bankDetailService);
    }

}
```

Let's see: well, there are many ways we could achieve this, but for the sake of brevity, we will stick to just 2 approaches.

# First Approach
We create the class file dynamically using ByteBuddy and save it to the disk, somewhere in my classpath. I do this before Spring starts scanning the classpath and creating beans. When Spring starts scanning my classpath, the newly created classfile is already available to it. If I decorate that class with the correct annotations, Spring will pick it up and do the rest.

## The right life-cycle hook
The challenge really is to find the correct life-cycle hook, so that we can create this class in the right phase. I found that the [EnvironmentPostProcessor](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/env/EnvironmentPostProcessor.html) to be the right thing. This is actually used to customize Environment variables, so this is a bit of a hack, but that's OK. It is going to be invoked right after the Environment is created, but before any beans are initialized. Looks like just the thing for us.

So, we go ahead and create our own __DynamicControllerPostProcessor__, which implements __EnvironmentPostProcessor__. Remember to create a __META-INF/spring.factories__ file and put the below entry there for this class to be invoked:

```property
org.springframework.boot.env.EnvironmentPostProcessor=com.swayam.demo.springbootdemo.dynamicclassesbytebuddy.config.DynamicControllerPostProcessor
```

Since our Loggers are not yet initialized, remember to use plain *System.outs* instead of Loggers.

This is how __DynamicControllerPostProcessor__ looks like:

```java
@Order(Ordered.LOWEST_PRECEDENCE)
public class DynamicControllerPostProcessor implements EnvironmentPostProcessor {

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
	     // put the code to create dynamic class here
    }
}    
```

## Creating the class dynamically
Here is the snippet for creating our cool new class:

```java
private void createDynamicController() {

String className = BankDetailController.class.getName() + "V2";

System.out.println("Creating new class: " + className);

Unloaded<?> generatedClass =
new ByteBuddy().subclass(BankDetailController.class)
  .annotateType(AnnotationDescription.Builder.ofType(RestController.class).build(),
    AnnotationDescription.Builder.ofType(RequestMapping.class)
      .defineArray("value", new String[] { "/v2/bank-item" }).build())
  .name(className).make();

Loaded<?> loadedClass =
generatedClass.load(getClass().getClassLoader(), ClassLoadingStrategy.Default.INJECTION);

try {
  Map<TypeDescription, File> map = loadedClass.saveIn(new File("target/classes"));
  System.out.println("Successfully saved the newly created class in: " + map);
} catch (IOException e) {
  throw new RuntimeException(e);
}

}
```

Notice, how we begin by instructing __ByteBuddy__ to *subclass* the __BankDetailController__. Next, we create the *@RestController* annotation using the __AnnotationDescription.Builder__. Then, we create the *@RequestMapping* annotation, and pass the *String[]* argument by using the *defineArray*. Finally, we add the *className*, which is a fully qualified className with the right package.

This class is then loaded into the ClassLoader and duly saved on the disk. The assumption, of course is that the *target/classes* is on my classpath.

# Testing with CURL
Boot up the [ReactJS Client](https://github.com/paawak/blog/tree/master/code/reactjs/library-ui-secured-with-google) and browse to <http://localhost:3000/>. You would see the login screen. Proceed to login with Google. Now hit *F12* to open the *Developer Console*. From the Menu, select Book -> Add New.

![Google login screen](../assets/2021/01/obtaining-google-access-token.png)

As shown above, you would be able to see the *Authorization Header* copy that. The curl command would be:

    curl -v -H "Authorization: MySecretToken" "http://localhost:8000/genre"

# Deploying on Apache

When deployed on Apache Server, the Authorization Headers cannot be read, happened with me. The solution is, in the .htaccess file, put the below lines:

    SetEnvIf Authorization "(.*)" HTTP_AUTHORIZATION=$1

# Sources
The Frontend code can be found here: <https://github.com/paawak/blog/tree/master/code/reactjs/library-ui-secured-with-google>.

The PHP Backend code can be found here: <https://github.com/paawak/blog/tree/master/code/php/php-rest-service-google-oauth2>.
