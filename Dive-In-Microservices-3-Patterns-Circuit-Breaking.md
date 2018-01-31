Before we delve deeper into circuit breaking pattern let us understand couple of patterns which will help us understand it better.

**Timeouts**

A timeout is an incredibly useful pattern while communicating with other services or data stores. The idea is that you set a limit on the response of a server and, if you do not receive a response in the given time, then you write a business logic to deal with this failure, such as retrying or sending a failure message back to the upstream service. A timeout could be the only way of detecting a fault with a downstream service. However, no reply does not mean the server has not received and processed the message, or that it might not exist. The key feature of a timeout is to fail fast and to notify the caller of this failure.

There are many reasons why this is a good practice, not only from the perspective of returning early to the client and not keeping them waiting indefinitely but also from the point of view of load and capacity. Timeouts are an effective hygiene factor in large distributed systems, where many small instances of a service are often clustered to achieve high throughput and redundancy. If one of these instances is malfunctioning and you, unfortunately, connect to it, then this can block an entirely functional service. The correct approach is to wait for a response for a set time and then if there is no response in this period, we should cancel the call, and try the next service in the list. The question of what duration your timeouts are set to do not have a simple answer. We also need to consider the different types of timeout which can occur in a network request, for example, you have:

Connection Timeout - The time it takes to open a network connection to the server

Request Timeout - The time it takes for a server to process a request

The request timeout is almost always going to be the longest duration of the two and I recommend the timeout is defined in the configuration of the service. While you might initially set it to an arbitrary value of, say 10 seconds, you can modify this after the system has been running in production, and you have a decent data set of transaction times to look at.

**Back off**

Typically, once a connection has failed, you do not want to retry immediately to avoid flooding the network or the server with requests. To allow this, it&#39;s necessary to implement a back-off approach to your retry strategy. A back-off algorithm waits for a set period before retrying after the first failure, this then increments with subsequent failures up to a maximum duration.

Using this strategy inside a client-called API might not be desirable as it contravenes the requirement to fail fast. However, if we have a worker process that is only processing a queue of messages, then this could be exactly the right strategy to add a little protection to your system.

**Circuit breaking**

We have looked at some patterns like timeouts and back-offs, which help protect our systems from cascading failure in the instance of an outage. However, now it&#39;s time to introduce another pattern which is complementary to this duo. Circuit breaking is all about failing fast, it is a way to automatically degrade functionality when the system is under stress.

Let us consider an example of a frontend example web application that is dependent on a downstream service to provide recommendations for services user can use. Because this call is synchronous with the main page load, the web server will not return the data until it has successfully returned recommendations. Now you have designed for failure and have introduced a timeout of five seconds for this call. However, since there is an issue with the recommendations system, a call which would ordinarily take 20 milliseconds is now taking 5,000 milliseconds to fail. Every user who looks at a services is waiting five seconds longer than usual; your application is not processing requests and releasing resources as quickly as normal, and its capacity is significantly reduced. In addition to this, the number of concurrent connections to the main website has increased due to the length of time it is taking to process a single page request; this is adding load to the front end which is starting to slow down. The net effect is going to be that, if the recommendations service does not start responding, then the whole site is headed for an outage.

There is a simple solution to this: you should stop attempting to call the recommendations service, return the website back to normal operating speeds, and slightly degrade the functionality of the profile page. This has three effects:

- You restore the browsing experience to other users on the site.

- You slightly degrade the experience in one area.

- You need to have a conversation with your stakeholders before you implement this feature as it has a direct impact on the system&#39;s business.

Now in this instance, it should be a relatively simple sell. Let&#39;s assume that recommendations increase conversion by 1%; however, slow page loads reduce it by 90%. Then isn&#39;t it better to degrade by 1% instead of 90%? This example, is clear cut but what if the downstream service was a stock checking system; should you accept an order if there is a chance you do not have the stock to fulfill it?

So how will it work work?

Under normal operations, like a circuit breaker in your electricity switch box, the breaker is closed and traffic flows normally. However, once the pre-determined error threshold has been exceeded, the breaker enters the open state, and all requests immediately fail without even being attempted. After a period, a further request would be allowed and the circuit enters a half-open state, in this state a failure immediately returns to the open state regardless of the error threshold. Once some requests have been processed without any error, then the circuit again returns to the closed state, and only if the number of failures exceeded the error threshold would the circuit open again.

Error behaviour is not a question that software engineering can answer on its own; business stakeholders need to be involved in this decision. When you are planning the design of your systems, you talk about failure as part of your non-functional requirements and decide ahead of time what you will do when the downstream service fails.
