# Microservice Architecture
I have attached my documentation for how I understand microservice it's not a holy book so feel free to argue about and correct me.

## what is a microservice:
  - is a physically deployable piece of software.
  - microservice is independent, loosely coupled, and focused on single functionalities or business capabilities.
  - microservices can communicate using various methods, including RESTful APIs, gRPC, or messaging systems.
    - use :
      - Synchronous Communication: `Typically handled by RESTful APIs or gRPC for real-time, request-response interactions.` 
        - RESTful-APIs (HTTP): Necessary for real-time responses and direct service interactions, such as payment processing in booking systems.
        - gRPC (HTTP/2): Best for high-performance, low-latency communication needs, such as real-time bidding systems in online advertising.
      - Asynchronous Communication (Event-Driven):  `Typically used for decouple services and handle events without immediate responses`
        - Using tools like RabbitMQ, where services publish and subscribe to events without waiting for immediate responses. Used in Decoupling services to enhance scalability and resilience.
      - Event Streaming:`typically used for continuous data processing and real-time analytics`
        - Using platforms like Ap ache Kafka or Apache Pulsar to handle continuous streams of data and real-time processing. Used in Real-time analytics, monitoring, and processing of high-throughput data streams.
  - changes in a microservices architecture are treated as first-class-citizen.
  - microservice architecture has the mindset of change built into the architecture and is expected as part of the architecture which can evolve in really incremental continuous way:
    - **What we end up with**:
      - ==> Changes are possible/expected and not expensive anymore.
      - ==> But it's operationally complex because we have to automate all the machines/containers creation  service discovery, logging, monitoring, fail recovery ..., and other stuff in engineering practices.

## Real Benefits of this style of architecture : 
  - better to keep those in mind to not end up with a microservice architecture with no benefits:
    - ### Asynchronicity:
      - for user experience :
        - Prefer timely partial over slow complete
      - return optimized complete:
        - for ranking/aggregation, not-display-end-user-consumer .  optimized complete data for ranking or aggregation is important for backend processes, but not necessarily for end-user display, where timely partial results might be preferred for better user experience.
    -  ### Business-Real-Benefit:<br>
     not like monolithic(risk of release is binary) having the right engineering practices in place, a large the number of services you have, you drive your risk of release down below. how?!  in the example below . 
      - Evolutionary Architecture of MS:
        - Solving complect(when u release component --> release feature to users at same time) Deployment:
          - Challenge :
            - Deploying a new component and releasing its features simultaneously can be risky and complex.
            - this takes more risk then necessary .
          - solution :
            - separate these 2 things :
                - deploy things and make sure they work operationally correctly and turn on features later, then you are incurring less risk doing that.
                - how this happens in MS :
                  - component are deployed into the production ecosystem very frequently --> continuously deployed(here comes the DevOps pipelines).
                  - MS deployed still has no impact yet because features are still inside it and at some point of time later you can expose its feature by turning on feature toggles.
                  - run new MS with almost identical capabilities with the currently in-use MS side by side , but still not impacted users yet by chosing the routing.
                  - doing that `enables do things like A/B testing` by routing for example 20% of users to this new feature
                  - also other services that needs that capability can `slowly migrate to the new enhanced version`
                  - `disintegration of old service` when all users/services are migrated to the new one thx to (operations and DevOps that lets us know when no one is routing to the old ms)
## Engineering Practices Necessary For This Architecture to Work: 
  - ### Design for Failure:
    -
  - ### Monitoring:
  - ### Aggregations Monitors:
  - ### Synthetic Transaction:

## Rule  of thumb:
  - It is fine to have a hybrid (mono-microservice) system, where some modules remain monolithic to minimize costs when they do not benefit significantly from being split into microservices.
  - Note:
    - Hybrid systems can be cost-effective, They should still be carefully evaluated for potential integration and complexity issues.
  - you can't excel in microservice architecture if you don't understand (Domain driven architecture/bounded context), DDD helps identify and define bounded contexts, which are foundational for effective microservice decomposition.
  - Architecting GreenField project :
    - Since the system is expected to grow and evolve, starting with a mono-repository can facilitate easier restructuring and rearchitecting without significant overhead.
    - why: because your system will definitely grow since you decided to use microservices architecture in the first place unless you have made the wrong choice. and there is no clear view of the system is future solution because is still young and evolving. Using a <b>mono-repositor</b>y can be beneficial. It simplifies the initial development and integration process. As the system grows and matures, having all the microservices in a single repository can make refactoring and architectural changes easier.

## Statements about ms, not 100% correct:
  - **Each microservice should have its own database** :(Partially True)
    + ==> `Each bounded context should have its own database`, not necessarily each microservice.</b>
  - **Data Consistency matters 100%**:(Partially True)
    - No sometimes data consistency does not matter and better to avoid it to not over complex your microservices.
    - only use it when it matters most :
      - Ex (when it does not matter ): you are collecting data for analytics or forecasting losing a few data will not matter.
      - Ex (when it matters ): accounting, banking, finance, orders, payment, database recovery logging, etc ... 

## A bounded context is not equivalent to a microservice:
  - ## ## Review on Bounded Context: 
    - A bounded context in Domain-Driven Design (DDD) is a logical boundary, while a microservice is a physical deployment unit. Although they often align, they are not the same.
  - ## In many cases : 
    - one bounding context => implemented as exactly one microservice.
  - ## Splitting a bounding Context (common as well):
    - There are technical reasons to split a bounded context into multiple deployable units   microservices.
    - ex:
      - separating: online services from housekeeping jobs that are running periodically.
        
## Microservice is it needed. Ask yourself :
  - ### Should that be a microservice:
    - **1- Multiple rates of change**:
        - Do parts of your system need to evolve at different speeds or in different directions? I(yes=> ms)
        - ex: (E-Commerce) :
          - cart and inventory are rarely touched, but we often like to improve our recommendation functionalities and optimize search functionalities.
    - **2- Independent lifecycles** :
      - Does the module should have an independent lifecycle, its own pipeline(ci/cd) from build to production? ==> (yes ==> microservice) 
    - **3- Independent scalability** : 
      - if load or throughput characteristics of parts of the system are different ==> they may have different scaling requirements (avoid scaling the whole monolithic by converting them to microservices)
    - **4- Isolated failure** :
      -  we want to insulate our app from a particular type of failure.
      -  Ex:
        - Your app depends on an unreliable third-party service with poor performance, it is better to have a microservice that directly interacts with that service failure rather than the whole application.
        - therefore we can create a fail-over mechanism to mitigate the whole failure of the app.
    - **5-Simplify interactions with external dependencies** :
        - AKA ( the Façade pattern )
        - very similar to "isolated failure", but this time the focus is on protecting our system from external dependencies that change frequently.
        - How:
          - Microservices can act as <b>a layer of indirection</b> to insulate you from a third-party dependency. Rather than directly calling the dependency, we can instead place an abstraction layer (that we control) in between the core app and the dependency. Further, we can build this layer so it’s easy for our application to consume, hiding the complexity of the dependency. If things change in the future—and you have to migrate—your changes are just limited to the façade, rather than having to do a larger refactoring.
    - **6-right tech for the job** :
      - Better to keep this to a minimum and only use it when required.
      - Teams should only use reliable technology with long-lasting support not just cool stuff that they want to try (google have 40000-engineers and only uses 8-languages).
      - ex :
        - keeping a reliable legacy system while adding on the new microservice with better technology.
        - using Python for IA and data science tasks, java for critical apps, and C++/Rust for low-level solutions.
        - if you have a team of 6-engineers, it is better to stick to one technology.
        - use this wisely to avoid the reverse effect.
## What set of problems should we consider microservices :
  - ### Maximize Easy-Evolution at the Architecture Level:
    - One of the unique things that ms-architecture gives you is  the idea of being able to maximize `easy evolution at the architecture level` it's like Lego-ability by taking one microservice to replace it with another microservice and no one notices that that happens. ( This is always true  if the feature is also present in the frontend and the frontend is as well a plugin-style or microservices style or the MS is a backend-and-not-frontend-related).
  - ### Make sure To avoid a single point of failure :
    - you may build a microservice but it actually have a single point of failure or in an other world a single distributed point of failure ex: an apigateway may fail if one of your service have a long timeout policy and the gateway exhaust its all resources or you will scale unnecessarely .
    - make sure your domain business is really independent where its decoupled in a way where our most basic use cases that have a direct relationship with consumers are absolutely decoupled and handle failure gracefully and are not down because of aggregation relation with other microservices.
  - #### Avoid developers-production toxic relationship :
    - no more projects ==> talking about products you build your product you deploy it.
  - ### Data consistency is difficult in Ms-Architecture :
    - **ACID nature in Monolothic is not the same in microservices**:
      - must consider how to deal with your distributed transactions in case of aggregation.
      - your ACID transaction nature will only work for local databases.
      - must consider that most databases are not thread if multiple services have write access to it.
    - it's possible to write to a database and make another service tightly related read from db, but this will make the other microservice tightly coupled to your db schema and naming. You better get data from API requests or messaging if you're not in a hurry and async is ok.
  - #### Enterprise-service-bus only used as dump pipes no logic:
    - It is safe to ignore and convert your service-oriented to microservices
  - #### microservices avoid service buses:
    - Modern microservice architectures generally recommend avoiding ESBs because they can introduce tight coupling and become bottlenecks. Smart endpoints and dumb pipes (simple communication mechanisms like HTTP or messaging) are preferred.
  
## When to avoid considering microservices :
  - #### Enterprise service-bus uses logic:
    - requires consideration (tight coupling issue)
    - why: information that is included in the service-bus is really part of the bounded context and you're violating that bounded context philosophy of microservices by trying to take some of that context, and externalizing it in another moving part when we're trying to break the coupling between the moving part in the architecture here to create the shared "nothing kind of architecture".

## Principals Of Microservices:
  - ### Modelled around Business Domain:
    - modeling things around business domain leads to creating services that have more stable APIs-boundaries.
    - It is easier to reuse them in different ways for different user interfaces.
    - makes teams that own them not only experts in the domain business themselves, but also the service ownership.
    - avoids lots of cross-cutting changes.
  - ### Culture of automation :
    - embracing a culture of automation is key to allowing you to manage all these multiple different services that are flying around  (manual microservices are unmanageable after a few microservices)
  - ### Hiding Implementation Details :
    - this is very essential if you want to evolve the internal of one service  without breaking others (related to the idea above of  creating services that have more stable APIs-boundaries).
  - ### Decentralise All the Things(avoids middleware):
    - avoids smart middleware(use dump-pipes & smart-endpoints only  ) see if u can push decision-making into your team , lower the barriers to entries for teams to look after and manage things themselves .
  - ### Deploy things Independently(Golden-Rule):
    - this is what u need to aim for .
    - you need to be able to make a change to a service and deploy it into production in isolation of everything else ==> if you can do that reliably then you're in a very good place.
  - ### Consumer-First (end-users b2c|b2b comes first):
    - place your consumer first, it's a very soft thing and services exist to be called so `think outside in`. solutions are built for customers not for developers to enjoy doing what is convenient for them. 
  - ### Isolate Failure (Critical):
    - Make sure you understand where the sources of failure are.
    - Every single communication between one service and an other is a `potential place where something can go wrong` ==> Plan for that , understand it , and know what you re going to do about it as a result.
  - ### Build Systems to be Observable(Critical):
    - Make sure to build your system to be observable .
    - Logs are very important and monitoring is so hard in Monolithic and almost impossible in microservices so make sure to do the necessary to make it possible.
    - building things like correlation-IDs, and aggregating stuff in a consistent and standard way is very important for debugging to be possible-easier and tracking to have meaning.
    - A microservice without proper monitoring is lost in a microservice architecture since it is untrackable when the solution becomes complex.
    
## Data consistency is difficult in Ms-Architecture :
  - achieving data consistency in microservice is quite complex, it requires a careful though 
    
