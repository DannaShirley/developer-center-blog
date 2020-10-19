---
title: Integrating with the Cloud
tags: Genesys Cloud, Developer Engagement
date: 2020-10-19
author: john.carnell@genesys.com
---

# Introduction

Integration.  It's not the most fun part of software development, but it is one of the more critical things you need to think about as you are looking at purchasing a cloud-based product.  Cloud products are great in the sense that you don't have to manage the infrastructure yourself, you get new features quickly and you are able to convert a fixed capital cost over to a variable expense (for the MBAs's in the audience).  However, the success or failure of your cloud acquisition is going to be tied to how successful you are in integrating your new cloud platform into your overall business processes/workflows and the IT ecosystem of your organization.  You need to understand that there are different styles of integrations and every cloud provider will give you different technologies to implement these categories. 

This article is going to walk through what different styles are available for cloud integration and talk specifically about which Genesys Cloud technologies are available for each integration style.  

:::Some thoughts on integration styles
The integration styles defined in this article are meant to be a very broad-based definition of integration types.  I have not many any direct attempt to tie these broad styles into the more traditional enterprise integration patterns found in Gregor Hohope and Bobby Wolfe's seminal book on integration: [Enterprise Integration Patterns](https://www.amazon.com/o/asin/0321200683/ref=nosim/enterpriseint-20).  The patterns defined in this book are still very relevant, but tend to be more fine-grained and focused on integration patterns of systems within the four-walls of an organization.  If you are interested in finding out more about these patterns, I also recommend visiting the [companion web-site](https://www.enterpriseintegrationpatterns.com/) for the book.  
:::

There are three broad styles of integration.  These three styles are defined by what you are trying to accomplish:

1. **Behavioral**.  I want Genesys Cloud to do something on behalf of my user or process.  For example, when the user clicks on a button in my home-grown CRM, I am going to call a Genesys Cloud API to save some data and carry out a set of business logic.

2. **Data**.  I want to move data in and out of Genesys Cloud.  Examples of this include: I need to pull all of the days conversations out of Genesys Cloud for my contact center and synchronize that data with my local data warehouse or I am building a near-time alerting dashboard for shift supervisor.

3. **Presentation**. I want to embed my application within Genesys Cloud or I want part of Genesys Cloud to be embedded within my application.  I want to integrate call-controls into my home-grown CRM so agents can take and receive calls right within the CRM.

Let's walkthrough these styles in more detail.

## Behavioral Integration Style

The behavioral integration style is when you need to integrate programmatically with your cloud provider so that the provider will execute some action on your behalf.  These actions are characterized as:

1. **Focusing on having the cloud provider do something for you**.  With a behavior integration style, you are usually asking a cloud provider to take some action on behalf of an individual user or a process.  For example, you might want to call a cloud-provider to save a piece of data and apply business rules along with it or doing a look up of a customer record.

2. **Being short lived in nature and point-to-point**.  Behavioral integrations are application-oriented and tend to favor single calls out to an API that return very quickly from a call.  They are considered point-to-point because the your application(s) are directly calling an endpoint on the cloud provider without a middle-man (e.g. a queue or service bus disintermediating the call)

3. **Are transactionally focused, rather then data synchronization focused**.  One thing that many people find confusing is that APIs can be used for transactional actions as well as for data synchronization.  Behavioral integrations styles are not data synchronization focused.

Genesys Cloud Offers three different mechanisms for carrying out behavioral integration styles.  They include:

1. REST-based APIs
2. Data Actions
3. Webhooks

### REST-based APIs

Genesys Cloud is an API-first platform.  All of the business logic, workflows and processes carried out in Genesys Cloud is exposed as REST-based web services.  The Genesys Cloud UI is built on the same APIs that we expose to our customers.  We offer a very wide variety of APIs to carry out common business tasks and we even have some customers who have completely forgone the use of the Genesys Cloud UI and instead have built their applications on Genesys cloud.  Most of our API's are built around a synchronous request-response model.  There are several things you need to keep in mind when looking at using our API.  First, you are integrating over the internet using a distributed web protocol.  You need to take into account that failures can occur and you need to build into your application common API invocation best practices like timeouts, retry, circuit breakers, bulkheads, fallback calls, caching when calling Genesys Cloud APIs. 

//TODO  Convert this to a time out
:::On building distributed applications
Cloud-based applications are distributed applications and the integrations built for them must take failure into account.  If you are not familiar with the concepts of timeouts, retry, circuit breakers, bulkheads, etc...., I highly recommend you take a look Michael Nygard's book [Release It!: Design and Deploy Production-Ready software](https://www.amazon.com/Release-Design-Deploy-Production-Ready-Software-ebook/dp/B079YWMY2V/ref=sr_1_2?dchild=1&gclid=CjwKCAjwz6_8BRBkEiwA3p02Vf8U0Mcz2-JXH9NlvRRKU5UE_NzqqEl879lei8Ln9opK-wZBaAld2BoC_loQAvD_BwE&hvadid=250075996547&hvdev=c&hvlocphy=9009669&hvnetw=g&hvqmt=e&hvrand=6812719388737011366&hvtargid=kwd-404904995006&hydadcr=22562_10354930&keywords=release+it+2nd+edition&qid=1603059694&sr=8-2&tag=googhydr-20) 

Michael covers the challenges of building distributed systems in detail and also many of the techniques I have referenced earlier.
:::


The second thing you need to consider is how much quickly you are going consume the Genesys Cloud APIs and at what volume.  As a general policy, Genesys Cloud does not try to monetize the invocation of their APIs.  However, we do implement [API rate limiting](/api/rest/rate_limits.html) and [API fair usage](https://help.mypurecloud.com/articles/routing-usage/) policies to protect the overall integrity of the platform and ensure that Genesys Cloud consumers are consuming APIs responsibly.  Make sure before you undertake a new integration you understand:

1.  The criticality of the API call within your workflow and protect your application appropriately.
2.  The API invocation density of your integration.  How quickly will your integration invoke an API.
3.  The API invocation volume of your integration.  Will your integration inadvertently consume your API fair use for the month.

Additional information about the Genesys Cloud platform API can be found [here](https://developer.mypurecloud.com/api/). 

### Data Actions

[Data actions](https://help.mypurecloud.com/articles/about-genesys-cloud-data-actions-integration/) allow you to declaratively invoke a third-party web-service or even a Genesys platform API from within a Genesys Cloud script, flow dialer or  pre-call rule.  Genesys Cloud data actions are not invoked like the Genesys Cloud REST-apis. Instead they are configured through the Genesys Cloud UI and the mapping of the request and response for the invocation is performed through JSON-Path transformations.  Genesys Cloud offers a number of pre-defined data actions for third-party vendors including:  SalesForce, ZenDesk and Adobe.  In addition, if a pre-defined data action, you can configure a generic action to another data source (including APIs within your organization).

### Webhooks
There is a third-class of API-integrations supported within Genesys Cloud: web hooks.  A webhook is a user-defined callback where the a user of a cloud-based platform registers a HTTP-based (usually JSON based) web service that will be called by the cloud-service provider back into the user's custom platform.  This is the reverse of what normally happens so rather then user calling an API on the cloud-provider, the cloud-provider calls a web service within the user's data center.  Webhooks are most commonly used when you are integrating system to system and you want to post the results of a workflow over to another workflow.

Genesys Cloud uses webhooks in two places within the platform.  The first place is that Genesys Cloud exposes the ability to [post notification's](https://help.mypurecloud.com/articles/set-jenkins-integration/) about an external event to a group chat room within Genesys Cloud.  Some examples of this type of integration includes posting alerts from on-line alerting systems like [Pagerduty](https://help.mypurecloud.com/articles/set-pagerduty-integration/) or build notification results from platforms like [Jenkins](https://help.mypurecloud.com/articles/set-jenkins-integration/).

The second use for webhooks within Genesys Cloud is with our [Altocloud](https://all.docs.genesys.com/ATC/Current/AdminGuide/Overview) feature.  Altocloud provides analytics and predictive engagement capabilities within Genesys Cloud and uses webhooks to send the data it collects to your internal CRM or back office application.

## Data Integration Style

When I was early in my career, working a for a consulting company, my manager at a client site once said something very profound to me.  He said "Code changes, but data is forever.  You will spend a large amount of your career moving data from one spot to another."  That single comment has run true over the course of my entire career and reflects the reality of cloud-based integrations.  Most integrations are going to be how you are going to get data out of your cloud provider and feed it to other systems within your IT ecosystem.

There are a lot off different mechanisms for pulling data out of a cloud provider.  However, when we talk specifically about how we do it in Genesys Cloud, the topic of data integration centers around the API-based Genesys Cloud [Analytics API](/api/rest/v2/analytics/overview.html) and event-based [Notifications](/api/rest/v2/notifications/notification_service.html).  

### Polling for Data with the Analytics API

The Analytics API is not a monolithic API, but rather encompasses three levels of data detail:

1. [Observations](api/rest/v2/analytics/overview.html#instantaneous_observations_metrics).  The Observation APIs return information about the current state of objects within Genesys Cloud.  Observation queries are meant to be a point in time snapshot of what is going on in your contact center.  They offer limited filtering and no data-based queries.
2. [Aggregate](/api/rest/v2/analytics/overview.html#aggregate_metrics). The Aggregate APIs return a summarized view of call center data over time with the ability to collect the data in time buckets.  This API has a rich query language
3. [Detail](/api/rest/v2/analytics/overview.html#detail_record_metrics). The Details APIs offer a fine-grained level of detail of user and conversation details.  It acts as a ledger of all activity associated with conversations and users.


The `Observation` APIs are generally used for building real-time dashboards and this API type can be polled periodically to retrieve their data (within the limits of API rate limits and API fair usage).  The `Aggregate` APIs are really designed for "ad-hoc" query-based applications where you are doing some basic presentation of aggregate data. 

The `Detail` APIs is where new Genesys Cloud integrators tend to run into trouble.  The `Detail` APIs offer a rich amount of conversation and user data.  Since this an easy to code to API, many integrators try to use this API to do regular pulls of large amounts of data with the desire to sync that detail data back to their own data stores, data warehouses or data lakes.  Unfortunately, the `Detail` API is an extremely expensive API that depending on the querying being issued is prone to timeouts and when used in this manner can quickly consume your fair use API allocations.

The Analytics team does offer a asynchronous version of the `Detail` APIs, called the [Jobs](/api/rest/v2/analytics/overview.html#detail_record_metrics) endpoints.  These endpoints allow a developer to issue a query and have the API immediately return with a 202 HTTP status code.  The integrator can then periodically check back to see if the query is complete.  The `Jobs` endpoint is much more efficient is terms of computing resources, but the data it returns can be up to 24 hours old.

### Into the Event Stream
So how do you deal with situations where you have low-data latency data requirements?  Polling for this data using the Analytics API introduces timeout problems and using the `Jobs` endpoints means you can not retrieve the most current data.  Genesys Cloud offers an event-based message stream that will publish conversation and user detail messages at the time the event occurs.  There are two mechanisms for consuming these messages streams:

1. The Web Socket based Notification Services
2. AWS Event Bridge

The WebSocket-based [Notifications](/api/rest/v2/notifications/notification_service.html) service allows you to open a web socket to Genesys Cloud and subscribe to events as they occur.  Events will be published to the WebSocket.  This web socket does not guarantee message delivery and if the web socket is disconnected at the time message is published, it will be lost.  This puts the onus on you as the developer to maintain socket state and to build data recovery code if the socket becomes unavailable.  This is usually done by using the `Details` API to retrieve any data that was missed from when the web socket went down and when a new one was established.

//TODO Need a link to our documentation
The second approach for consuming events is using the AWS Event Bridge.  Using event bridge, notifications from Genesys Cloud are published to AWS and then forwarded to a target object within the customer's AWS account.  This target can be a Kinesis stream that can fire a lambda or write the data to S3.  While using AWS Event Bridge requires an AWS account, Genesys Cloud will attempt to deliver a message to the event bridge 7 times during a 24 hour period.  Once the message hits Kinesis, you as the integrator can configure your Kinesis stream to persist the data for up to 7 days.  

## Presentation Integration Style
## Closing thoughts
## Additional resources

Challenges of cloud integration
  - Bewildering amount of choices
  - Understanding usage patterns
  - Volume
  - Reliability
  - Vendor has multiple ways of integrating
  - Rapidness of change

    
  Data Integration
    Quote: Code comes and go, but data is forever.
    Really depends on the direction of data, size of data being pushed and pulled and the data latency.
    
    - Putting data into Genesys cloud is usually done with the APIs.  However, you need to be aware of how quickly you are putting data into the cloud with the APIs.  If you are looking at a large data load, talk with our architects
      and PS folks earlier in the process.
    - Pulling data out of the cloud 
      - First understand what type of data you are trying to look at.
        - Genesys Cloud has three types data APIs: Instantaneous Observations, Aggregations and Detail records
      - If you are looking at pulling data for dashboards that are not dependent on near-time access use the APIs.
      - Ad-hoc queries and data lookups use the APIs
    - Pushing data out of the cloud (e.g. Data Synchronization)
      - For large amounts of data with low data latency requirements - Use the jobs API
      - For near-time data or data where you can not have the latency with the Jobs API use one of two mechanisms:
        - Notifications API with Web Sockets
          - Notifications does not do guaranteed retry so if the socket breaks, everything goes does.  Fortunately, you can tell when a socket goes up or down and you can use the details API to pull the data missing from the timeframe.  It sucks, but living in a distributed development world is rarely rainbows and farting unicorns.  Not pub sub.  Its an non-persistent event stream, if you done consume the messages
        - Notifications with AWS Event Bridge
          - We are in beta now for AWS Event bridge.  Event Bridge will retry message deliver up to 24 hours.  Then you can hook Kinesis up and process and persist the message for as long as you want.
    
  Presentation Integration
    - UI Based Integration
    - Embedded an application directly within Genesys Cloud  (Look at our client apps )
    - Embedded a component within a page (e.g. chat)
    For genesys cloud you need to differentiate between agent and client side integration.

Closing thoughts
  - Understand what integration style you are trying to use and choose the right type implementation technology API.
  - Look at your options before jumping right to an API.  Including looking at connectors and third-party plugins. App Foundry. Also check out blueprints
  - Plan for failure
