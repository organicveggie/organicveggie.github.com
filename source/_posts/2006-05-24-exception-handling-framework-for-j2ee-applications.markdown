---
layout: post
title: "Exception Handling Framework for J2EE Applications"
date: 2006-05-24 23:00
comments: true
categories: 
 - J2EE
 - Java
 - Programming
 - Web Development
---
OnJava has an interesting [article](http://www.onjava.com/pub/a/onjava/2006/01/11/exception-handling-framework-for-j2ee.html) on generic approach to handling exceptions within a J2EE framework. The basic idea involves creating a generic base class for exceptions and a basic handler for all requests that includes dispatcher to pass the request along to the appropriate method.The fundamental idea is fairly sound and the idea of including a context to allow reuse of generic exceptions seems like a strong idea. 

My concern is that in a large project, the use of a "context" may make it difficult to figure out which specific message code is used in a given place. The generic exception handling is especially beneficial for the "this should never happen" cases where there isn't much the app can do about the exception, but should still handle it in a robust manner.
