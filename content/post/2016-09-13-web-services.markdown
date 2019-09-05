---
layout:     post
title:      "SOAP and RESTful: Which to choose?"
subtitle:   "Differences and tips of choosing web services "
date:       2016-09-13 12:00:00
author:     "Shengwen"
header-img: "img/home-bg-o.jpg"
tags:
    - Web Services
    - RESTful
---

**What are the differences between both SOAP WS and RESTful WS?**

The SOAP WS supports both remote procedure call (i.e. RPC) and message oriented middle-ware (MOM) integration styles. The Restful Web Service supports only RPC integration style.

The SOAP WS is transport protocol neutral. Supports multiple protocols like HTTP(S),  Messaging, TCP, UDP SMTP, etc. The REST is transport protocol specific. Supports only HTTP or HTTPS protocols.

The SOAP WS permits only XML data format.You define operations, which tunnels through the POST. The focus is on accessing the named operations and exposing the application logic as a service. The REST permits multiple data formats like XML, JSON data, text, HTML, etc. Any browser can be used because the REST approach uses the standard GET, PUT, POST, and DELETE Web operations. The focus is on accessing the named resources and exposing the data as a service. REST has AJAX support. It can use the XMLHttpRequest object. Good for stateless CRUD (Create, Read, Update, and Delete) operations. 
- GET - read()
- POST - create()
- PUT - update()
- DELETE - delete()
          
SOAP based reads cannot be cached. REST based reads can be cached. Performs and scales better.

SOAP WS supports both SSL security and WS-security, which adds some enterprise security features like maintaining security right up to the point where it is needed, maintaining identities through intermediaries and not just point to point SSL only, securing different parts of the message with different security algorithms, etc. The REST supports only point-to-point SSL security. The SSL encrypts the whole message, whether all of it is sensitive or not.

The SOAP has comprehensive support for both ACID based  transaction management  for short-lived transactions and compensation based transaction management for long-running transactions. It also supports two-phase commit across distributed resources. The REST supports transactions, but it  is neither ACID compliant nor can provide two phase commit across distributed transactional resources as it is limited by its HTTP protocol.

The SOAP has success or retry logic built in and provides end-to-end reliability even through SOAP intermediaries. REST does not have a standard messaging system, and expects clients invoking the service to deal with communication failures by retrying.

**How to decide what style of Web Service to use? SOAP WS or REST?**

In general, a REST based Web service is preferred due to its simplicity, performance, scalability, and support for multiple data formats. SOAP is favored where service requires comprehensive support for security and transactional reliability.

The answer really depends on the functional and non-functional requirements. Asking the questions listed below will help you choose.
- Does the service expose data or business logic? (REST is a better choice for exposing data, SOAP WS might be a better choice for logic).Do the consumers and the service providers require a formal contract? (SOAP has a formal contract via WSDL)
- Do we need to support multiple data formats?
- Do we need to make AJAX calls? (REST can use the XMLHttpRequest)
- Is the call synchronous or  asynchronous?
- Is the call stateful or stateless? (REST is suited for statless CRUD operations)
- What level of security is required? (SOAP WS has better support for security)
- What level of transaction support is required? (SOAP WS has better support for transaction management)
- Do we have limited band width? (SOAP is more verbose)
- What’s best for the developers who will build clients for the service? (REST is easier to implement, test, and maintain)

