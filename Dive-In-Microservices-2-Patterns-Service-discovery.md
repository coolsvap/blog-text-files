With monolithic applications, services invoke one another through language level methods or procedure calls. This was relatively straightforward and predictable behavior. As application complexity increased we realized that monolithic applications were not suitable for the scale and demand of modern software, so we moved towards SOA or [service-oriented architecture](https://en.wikipedia.org/wiki/Service-oriented_architecture). The monoliths were broken into smaller chunks that typically served a particular purpose. But SOA brought its own caveats in the picture with inter-service calls, SOA services ran at well-known fixed locations, which resulted in static location of services, IP addresses, reconfiguration issues with deployments to name a few.

**Microservices are easy; building microservice systems is hard**

With microservices all this changes, the application typically runs in a virtualized or containerized environment where the number of instances of a service and their locations can change dynamically, minute by minute. This gives us the ability to scale our application depending on the forces dynamically applied to it, but this flexibility does not come without its own share of problems. One of the main ones knows where your services are to contact them. Without the right patterns, it can almost be impossible, and one of the first ones you will most likely stumble upon even before you get your service out into production is service discovery.

With Service discovery, the services register with the dynamic service registry upon startup, and in addition to the IP address and port they are running on, will also often provide metadata, like service version or other environmental parameters that can be used by a client when querying the registry. Some of the popular examples of service registry are [Consul](https://www.consul.io/), [Etcd](https://coreos.com/etcd/). These systems are highly scalable and have strongly consistent methods for storing the location of your services. In addition to this, the consul has the capability to perform health checks on the service to ensure its availability. If the service fails a health check then it is marked as unavailable in the registry and will not be returned by any queries.

There are two main patterns for service discovery,

**Server-side service discovery**

Server-side service discovery is a microservice antipattern for inter-service calls within the same application. This is the method we used to call services in an SOA environment. Typically, there will be a reverse proxy which acts as a gateway to your services. It contacts the dynamic service registry and forwards your request on to the backend services. The client would access the backend services, implementing a known URI using either a subdomain or a path as a differentiator.

Server side discovery eventually runs into some well known issues, one of them being reverse proxy bottleneck. The backend services can be scaled quickly enough but it requires monitoring. It also introduces latency causing increase in cost for running and maintaining the application.

Server side discovery also potentially increases the failure patterns with downstream calls, internal services and external services. With server side discovery you also need to have a centralize failure logic on server side, which abstracts most of API knowledge from client, handles failures internally, keeps retrying internally and keeping client completely distant till its a success or catastrophic failure.

**Client-side service discovery**

While server-side service discovery might be an acceptable choice for your public APIs for any internal inter-service communication, I prefer the client-side pattern. This gives you greater control over what happens when a failure occurs. You can implement the business logic on a retry of a failure on a case-by-case basis, and this will also protect you against cascading failure.

This pattern is similar to its server-side partner. However, the client is responsible for the service discovery and load balancing. You still hook into a dynamic service registry to get the information for the services you are going to call. This logic is localized in each client, so it is possible to handle the failure logic on a case-by-case basis.
