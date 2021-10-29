---
layout: post
title: Google Authentication with Spring Boot Backend
author:
  display_name: paawak
  login: paawak
  email: paawak@gmail.com
  url: ''
author_login: paawak
author_email: paawak@gmail.com
date: '2021-10-26 09:44:00 +0530'
categories:
- java
tags:
- java
- auth
---
# Background
I was trying to secure my Spring Boot based REST Endpoints using Google Authentication. I was not able to find a satisfactory example online. Most of the examples I found would use the classic OAuth2 Pattern, wherein, the user is redirected to Google Authentication Page, the user enters the credentials and then, he is redirected back to the actual site. This, in my mind, is very UI-centric.

This is not the pattern I had in mind. I wanted to build a pure REST solution. I wanted to solve the use case where the user has already obtained the OAuth2 token, and now, he is using that token in the Header with each REST call. All the backend needs to do is to validate whether this Header is a valid Oauth2 Token and extract the Name, Email and Roles of the Principal from it. If there is no Token or the Token is invalid, the call with simply fails with a 401 error.

If you think about the implementation, from a high level, of course, it has to be some kind of a Filter, which is invoked for a set of REST Endpoints that we are securing. This Filter will do the validation of the Authentication Token. The trick is: how to tie this all up with Spring Boot or rather Spring Security?

# Getting Started
You need to do some ground work to register in the Google Credential Console before we can proceed. I have described that in my previous post, [Google Authentication with ReactJS and Typescript](https://palashray.com/google-authentication-with-reactjs-and-typescript/).

Next, we will develop a simple Spring Boot REST application that works with the above Frontend ReactJS Code. The application that we are creating will have the same set of REST Endpoints as the one described here [Google Authentication with PHP REST Server](https://palashray.com/google-authentication-with-php-rest-server/).

# Implementation
## Some Concepts
We need to use a AuthenticationFilter. And then we will customize it. We will use our won AuthenticationManager, which will help us to extract the Principal from the Header Token. We will also use a custom AuthenticationSuccessHandler, which will pretty much do nothing. The SavedRequestAwareAuthenticationSuccessHandler, which is the default for the AuthenticationFilter will not work for us, as it would error out further in the filter chain. We will use the SimpleUrlAuthenticationSuccessHandler instead and then set our own RedirectStrategy, which will, again do nothing.

```
2021-10-29 10:46:50.296 ERROR 8420 --- [nio-8000-exec-7] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.IllegalStateException: Cannot call sendError() after the response has been committed] with root cause

java.lang.IllegalStateException: Cannot call sendError() after the response has been committed
	at org.apache.catalina.connector.ResponseFacade.sendError(ResponseFacade.java:472) ~[tomcat-embed-core-9.0.53.jar:9.0.53]
	at javax.servlet.http.HttpServletResponseWrapper.sendError(HttpServletResponseWrapper.java:129) ~[tomcat-embed-core-9.0.53.jar:4.0.FR]
	at javax.servlet.http.HttpServletResponseWrapper.sendError(HttpServletResponseWrapper.java:129) ~[tomcat-embed-core-9.0.53.jar:4.0.FR]
	at org.springframework.security.web.util.OnCommittedResponseWrapper.sendError(OnCommittedResponseWrapper.java:116) ~[spring-security-web-5.5.2.jar:5.5.2]
	at javax.servlet.http.HttpServletResponseWrapper.sendError(HttpServletResponseWrapper.java:129) ~[tomcat-embed-core-9.0.53.jar:4.0.FR]
	at org.springframework.security.web.util.OnCommittedResponseWrapper.sendError(OnCommittedResponseWrapper.java:116) ~[spring-security-web-5.5.2.jar:5.5.2]
```

# Testing with CURL
Boot up the [ReactJS Client](https://github.com/paawak/blog/tree/master/code/reactjs/library-ui-secured-with-google) and browse to <http://localhost:3000/>. You would see the login screen. Proceed to login with Google. Now hit *F12* to open the *Developer Console*. From the Menu, select Book -> Add New.

![Google login screen](../assets/2021/01/obtaining-google-access-token.png)

As shown above, you would be able to see the *Authorization Header* copy that. The curl command would be:

    curl -v -H "Authorization: MySecretToken" "http://localhost:8000/genre"

# Sources
The sources for this example can be found here: <https://github.com/paawak/spring-boot-demo/tree/master/google-auth-with-spring-boot>

The Frontend code can be found here: <https://github.com/paawak/blog/tree/master/code/reactjs/library-ui-secured-with-google>.
