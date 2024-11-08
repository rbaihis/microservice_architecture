# Reseliance PatternsMust Know Building Microservices :
</br>

   - ## Brief Overview KeyPoints i deducted not all:
     ![](images/keypoints.png)
      

---
---
## Distributed-System Defenition
  * A distributed system is one in which the failure of a computer you didn't even know existed, can render your own computer unusable.

## If your system can't withstand production you're doomed:
  * if your business idea can't be mapped into a system available at least 99% then the system we built is useless. customers will run away.
  * $availability = upTime/totalTime =  \frac{\bar{\text{meanTimeToFailure}}}{\bar{\text{meanTimeToFailure}} + \bar{\text{meanTimeToRecovery}}}$

##  Key Things to Do to Mitigate Failure:
  * write more tests: unit, end-2-end, as well as (load-test, spike, resilience, [ToxyProxy_tool_to_delay_traffic_between_services_for_testing](https://github.com/Shopify/toxiproxy/blob/main/README.md))
  * try to recover from failure as fast as you can, we want to have a detection mechanism and ways we can backup as soon as possible.
***
***
***
## Patterns we will study :
![patterns we will study](images/patterns.png)

 ### Resilience patterns (graceful Degradation & FallBack):
  * **What is a Resilient System**:(trying to do your best not to show the failure) is the ability of a system to handle **unexpected** situations `without users noticing`(best-case), or with a `graceful degradation` of service
---
  ### FallBack and Graceful Degradation
  - **FallBack design**?
    - The Fallback design pattern is a resiliency pattern used in microservices architecture to handle failures gracefully. It ensures that when a service is unavailable or fails, the system can continue to operate by providing an alternative response or executing a predefined fallback mechanism.
 - **The fallback method**:
   - mechanism works like a try/catch block. If a fallback method is configured, every exception is forwarded to a fallback method executor. The fallback method executor is searching for the best matching fallback method which can handle the exception. Similar to a catch block.
 - **FallBack API**:
   - A fallback using API Gateway is a plan B for an API - when the primary API service fails, the API Gateway can redirect traffic to a secondary service or return a predefined response.
### Example of Graceful degradation and FullBack (Read-problems) :
  - **Problem1**: trying to retrieve data from a system that is down ==> you could serve an older value and hope nothing can go wrong importance of caching.
  - **Problem2**: can't access user recommendation ==> return lower quality as recommendation per country, city, or gender better than nothing.
  - **Problem3**: search optimiser not working ==> call a slower system, for example, search in sqlDb when ElasticSearch is down better than nothing.
  - **Problem4**: can't send result immediately ==> notify by we will send you the result later in an email when we're done.
### Example of Graceful degradation (Write-problems) :
 - **Problem5**: command to change data.
   - sol 1: **outbox table pattern**: insert it in DB of the caller service the request + schedule a persistant retly until successful
   - sol 2: **send it as message**: instead of HTTP use asynchronous MQ approach and the system will handle it when available (handle end-user retry gracefully)
   - sol 3: **Log an ERROR**: at least log an error, that raises the alarm calling for manual intervention.
   - sol 4 : **Case: too frequent Errors**: send an error event to `supervisor for automatic Recovery([saga pattern]())`.


---
## Isolation (importance of foreseeing potential errors):
 ### Catastrophic Failures Examples :
  - Ex1 : **Long Chain of REST calls** : A->B->C->D->E (anything can go wrong in this chain)
    - any failure can blow up the whole thing.
    - any delay in this processing can delay all the upstream systems
    -  => its a terrible fragile architecture (simply a distributed monolith architecture)
  - Ex2 : **A request causes the instance to restart**:  and client keeps retrying or refreshing.
  - Ex3 : **A poison pill case MessageQueue**: receiving some type of particular message that causes the listener to crash.
     - and that message is retried again and agian not only just crushing your listener once .
     - => Freeze the entire stream of messages 
  - Ex4 : **Undounded Queues**: running of (disk-space/RAM) in messageBroker, instance crashes with outOfMemoryError.
     - reasons:
       - Not monitoring the size of their topics(Kafka-case)
       - No Alarms when 10 billion  of messages on the queues
       - => publishers and consumers services relies on the the brokers will fail as well.
  - Ex5 : **Concurrent Massive DB Import/Export overloads the DB**: end-users running db heavy import-export operation that takes too long and keep refreshing
     - => result on concurrent heavy operations that exhaust the database .
     - ==> its very important to introduce boundaries around parts of your system.
 ### Introducing Boundaries `BulkHead (aka Failure Units)`:
  #### Core Isolation pattern :
   - **Pure Design issue**: what should be independent of what ?
 ![draw_boundaries](images/boundaries.png) <br>
   - **1st** we need to identify the pieces in our architecture that should still work, even if other parts on our system are down .
   - **2nd** you should draw these lines and say whatever in green should not depend on stuff on blue or yellow.
   - **3rd** if the blue has to work reasanbly without the green ones , you would have to `cache(replicate) some data` of the green one to survive them being down (`use these boundaries as boundaries of redundancy`).
   - **4th** use the bundaries as **units of scale** enough to scale up that bubble and not every single service.
    
 ### Isolating Bulkhead Examples:     
  - by **Key Features**: catalog , search, checkout
    - The catalog could be down, but users can still search   and check out  products from their cart.
  - by **Markets|Region**:
    - deploy the same product in different countries, if you lose Romania you can still save France.
  - by **Tenants**:
    - if you develop cloud solutions to have your clients not influencing each other, having a VIP tenant should not fail if your regular customers are growing in numbers.
 ### To isolate them you Can Use:
  - **(connection|thread) pools**:
    - **within the** same application: first level, you could use different connections and thread pools, different resources within the same application.
  - **Application Instances**:
    - second level, deploy completely different application instances one process could fail the other could still work.
  - **Databases, Queues**:
    - next-level, will be to fully separate down to database and messaging infrastructure to have completely separated resources involved in each of the bulkheads. Depending on how much you want to invest in this and how and where do you see the failures you can go deeper or not.
    - ---
### `Throttling`:
 - throttling means you need to put an artificial limit to the amount of load the server could take.
 - **why limiting the Load** on a server:
   - **Prevent crash** We prefer to see `503 service unavailable better than outOfMemoryError` .
   - **Prevent Long ResponseTime** we prefer to have an error in 30second then success withing 1 minute , end user may consider it a failure and retry and overwhelm the service more with the same request that is already processed, and to make it worse if method called is not idempotence then dister will happens (time-response is based on context).
   - **Protect critical endpoints** services are not allowed to impact more important critical services endpoints.
     - ex: export invoices service that the secretary is running is **not allowed to impact the place order endpoint** you want to give more priority to the order service because its business-critical.
     - **ensure fairness** return 429ToManyRequestStatus to greedy clients/tenants to tell that client to backOff and lower their rate of call.
     - **Limit auto-scaling to fit the budget** of scaling on a cloud provider or your private cloud.
       
 - #### what to throttle, basically those 3:
   - **Request Rate** number of request/second , ex max 300 requests/seconds, eg via @RateLimiter
   - **Concurrency** number of requests that happen at the same-time(parallel), ex max 3 exports in parallel, eg via @BulkHead .
   - **Stuff that directly costs you** :
     - **Traffic** max 1Gb/minute .
     - **Resource units if you're in a cloud environment**  max 300 credits/minute.
 - #### Where to throttle these:
   - globally
   - per client
   - per endpoint
- #### example of throttling features :
   - ![throttling example](images/throttlingex.png) 
   - Feature A critical (untouched) business-critical can't be degraded.
   - feature C needed (but degraded) to give some extra space to feature A.
   - feature D (stopped completely) to leave its resource utilization to both A and C, so the system won't freeze/crash when reaching maximum capacity (case resource limitation/budget-tight).
   - ps: must consider optimization problems like linear programming to calculate the degradation rate in the pick of feature A.
     
- #### facing a lot of Usage Spikes:
  - you usage pattern looks like this: ![spikes](images/spikesInUsagePattern.png)
  - ==> bulks of requests coming and being paged to you like 1000req/sec how do you survive?
  - **First all Engineers must Understand** is the performance response curve.
    - ![performance Response Curve](images/performanceResponseCurve.png)
    - **throughput**: is the number of requests that can be successfully handled per unit per time.
    - **concurrent requests single machine** the number of concurrent requests that start executing in a single machine.
    - => the more pressure u put on a machine at some point you re going to see `degradation of response time` --> u gonna see a slowing down of the entire speed of that system.
      - why? because you will have:
        - Memory issues   
        - Compete on processor
        - `kill the database` the database will also be affected since too many write requests concurently can occure especially if ACID will increase response time.
     - **solutions**
       - **1-degradation-ok** `identity load too high` and enqueue some of it to assure no failure but degradation in response-time is ok and does not matter ==> ex [0 --> safeNotToCrush]
       - **2-Critical-responseTime** Identify load close to best performance when starts slowing down a bit to ensure good response time that will not affect customer experience endUsers/ClientsB2B ==> ex [0 --> endOfSweetSpot].
       - **trick to Improve Overall Performance(use Async)** for the excessive load ,  
           - **responsetime queue Monitoring** now the $resTime= queueWaiting + executionTime$` make sure that the time is not too much so optimize with consideration.
           - **queue sizes  Monitoring**  monitor queue size between services to not blow up queue with outOfMemory.
-  ### Bounded Queues(LatenctControl) relationship with throttling:
  - Bounded Queue is a part of the Latency Control pattern but its quit in a relationship with throttling in the isolation pattern when we decide to use optimization in our service when using throtlling.
  - it's important that queues are bounded to avoid the pitfalls of having a large queue, which may leadj to either having a longer response time or  the message queue itself blows up with outOfMemoryError. So its important to have a max of the number of element in a waiting  queue that your infra can handle and that your service can process later without very long response time.

     - ---
 - ###  Full Parameter Check input (between microservice):
   - **Important !** Any data that comes to your system has to be validated `before you invest  on any resources in it` to not do actions all from the start, that will fail eventually later when investing on resources.
   - **validate ClientRequests/APIsResponses** as soon as you see it to make sure the parameters are correct.
   - **Validate only What you care about** do not waist your time over validating the full schema just validate the fields your service care about `don't go too far with validating fields u don't care about`.
   - ==> **Robustness Principal** Be conservative in what you do, but be tolerant in what you accept from others.
     
---
 ## Lateny Control:
  - it's important to control your latency in a microservice architecture, using the best timouts, failingFast if failure is a fact and majority of request is failure use circuitBreaker, FanOut request to multiple services and get the quickest answer for expensive but some usecases .
  - ### Timeout:
    - when you call a system the first thing that you should put in place is TimeOuts.
    - **set a timeout every time u block** not only when you call network , perhaps you block to get a signle in a reactive sync or you do future.get()  ==> so `any time you block you should have timeout there`.
    - **TimeOut Too Large**: Impacts my response Time --> impact my Client
      - mind the default timeout in frameworks some are 1-minute, and some even **unbounded** `your call may be blocked forever` unnormal.
    - **TimeOut Too Short**:
      - `May result in False Errors`  the operation may succeed Later on the server and you didn't know.
    - **How To Decide**
      - for `TimeOut Too Short` Keep above the API measured/SLA response Time above what they call maximum a bit (max it a bit or 99%th) to avoid false errors and to be safer.
      - `Tailor per Endpoint` ex longer for Export operation or functionalities with long execution time.
      - `Monitor + Alarms` monitor timeouts to see for any issues , some Apis may change their implementation and they don't notify us about it better safe then sorry.
      - ---
  - ### FAN-OUT & Quickest Reply
    - is firing **multiple requests at the same time** for the `same goal` and have the fastest response win .
    - used in mission-critical when u need a very fast response .
    - in some domain its cheaper and suitable for this to maintain business.
    - ---
  - ### FailFast(latency-control)--Retry(isolation)--idempotency(loose-coupling):
    - `we have call failed or timed out` what i can do ?
      - I want to **`Retry(isolation)`** :
        - check **Cause** :
          - 400 --> your fault client is fault bad request fix it .
          - 500 --> you have no clue. (if repeated more then 95% of time just circuit-Break and do'nt use it until fixed)
          - **timeout** --> you could try to retry maybe you `misjudged your client timout`.
          - **Part of response {retryable:(false|true)}** --> if true means either `operation is idempotent` safe to retry and it's ok or a minor issue in the API then give it a try, but if > 99% same problem consider circuit breaker.
        - **Ask Yourself ?**:
          - **Worth retrying**, **Max attempts**, **What BackOff**, **How to Monitor** , is The operation **idempotent**.
        - `Retry !!` **Requires Operation to be `Idempotent(looseCoupling)`**.
        - **Idempotenet Operation** is an operation if repeating it `doesn't change or harm any state` of the server.
          - **Ex Idempotent ops**:
            - Get productByID, CancelPaymentById, UpdateProductPrice(entering the same input twice will result in same state).
          - **Ex Non-Idempotent ops**:
            - **PlaceOrder** retry could create a second order there fore it must be developed to be Idempotent.
              - **How to fix** (do strategies in your code to make it idempotent):
                - **1-detect duplicates via pastHoursOrder ex : pastHoursOrder= Map<CustomerId,List<hash(order)>> and answer u have ordered already within an hour is this a duplicate or u want to proceed.
                - 2-Alow the client to give you the Id, if the clients adds a unique identifier to the request and repeat it ==> you will just have UniqueKeyViolation if you try to persist the same order and good luck retrying.
                - ---
    - #### Case Consider Circuit Breaker (relation with RetryIsolation & IdempotencyLooseCoupling)
      - **Via Monitoring `over the last 5min 99% request to system Failed or timed out`**
        - **Just Fail Fast**
          - use the `the Circuit-Breaker` to cut off the service and fail fast `alert supervisor` to fix issue or fix timeout if client timeout is lower then the API is new config that it does not match the existing SLA, API provider may have changed the execution-time without notifying or he is overloaded.
          - ![circuit breaker](images/circuitBreaker.png)
  ### Where & When To implement These : 
   - ![When & where to implement these](images/whentoimplement.png)
   - when you re calling a `fragile/slow system`, you should discuss about those patterns very early, you are going to consider `timeout retry` for those.
   - when you have calls going between diffirent BulkHeads that you have identified in your architecture, if there is calls aggregation or calls you need to enforce these protections in between.
   - **Where To implement These**:
     - Most teams implements these patterns in the API-GateWay to protect their microservice ecosystem.
       - **Cons**:
         - finding many microservices under the gateway ==> possibility to fail as a whole if one microservice goes down.
       - **Solution for this**:
         - keep the number of microservices that an API gateway that it protects Low, those bulkhead should be smaller if you want to build more reseliant systems.
   - **Timeout-CircuitBreaker Implementation Example**:
     - **case MS communicate with external-API**:
       ![timeout external API case](images/timoutexternalapicase.png)
       - It's important to define your timeouts in outbound and inbound gateways if operations in your microservices rely on external API.
       - why, to mitigate the risk of changes in execution time or Latency of the external API, and correct that by only updating the   
     - **case Synchronous MS-->MS communication**
       ![timeout external API case](images/innertimeoutecase.png)
---
---
## Loose Coupling:
  - ### Stateless Service:
    - keeping state inside a service can impact `Consistency` `Availability` `scalability`
    - **Problems Keeping State**:
      - *Consistency* All nodes must sync it
      - *Availability* Failover nodes must replicate it 
      - *Scalability* New instances must copy it
    - **How To `make it Stateless`**:
      - **Move State Out To**:
        - **Databases** Keep state on db and make your services fetch the state
        - **Clients** put state `if not security critical` on clients (mobile, browser) and `if related to UI flow`. 
        - **Via Request Tokens** accept some of that state to be  coming in  as request tokens, `if user metadata` like email, roles, id
     - ==> By keeping your service stateless, then you can start many of them and `distribute loads between them`. ![](images/locationtransparency.png)
     - ---
  - ### CAP Theorem and Microservices:
    - {Consistency, Availability, Partionning} u can only have a subset of 2 element only either {C,A}{C,P},{A,P} and can't have all three.
    - microservice is by default `Partitioned`.
    - you need to chose between either {P,A}or{P,C}.
    - Microservices `Favors Availability` for scaling purposes.
    - ==> We need `to trade off Some Of the Consistency for it`, this could scare us because we don't know what the current state is right now, and also because we're used to the monolithic approach and its acid transactional databases.
   
  - ### Asynchronous Communication (MQ):
    - #### Why Async (MQ)
      - **Guarantee Delivery** message queues guarantee delivery even if the receiver is unavailable now, not like sync(REST) methods.
      - **Prevents cascading Failure between Bulkheads**:
        - Errors will not occur on the sender's side since he will not care. `Note! the double-edged sword in ACID or aggregation cases, must consider patterns to verify.
        - taking too long on the callee server will not impact us.
      - **good for isolating bulkheads** The reason is in the above lines, because u know that the job u assigned  for them will be treated sooner or later or reported.
      -  **Mindset required For Async** This calls for change in architecture, and think differently because we are most used to a naive approach async(Rest,...) approach when architecting usually and coding.
      -  **Async vs syncResilience_Concern(REST)** Both are scary and require foreseeing and a change in mindset.
        - in sync care in MS, such retry, circuit breaker idempotency make it scary and also produses some dependencies for the caller if a respose is critical .
        - Async care is exactly the oposit benifi of sync in term of ready response open complete or failure or timeout , u actually don't know when the other side will do his thing.
    - #### Mindset Of Async programming, different techniques, and change in architecture:
      - Generating IDS, Correlation IDs, and all sorts of kungfu techniques will come into play to achieve what you usually do simply in sync-mode.
      - **The caller should not need a response or status** This will lead to design change.
        - **Don't Expect that changes are applied instantaneously** state/data could be in transition in a data queue and not yet processed.
        - **Embrace/accept Event Consistency**:
          - ask yourself what will you lose if you go Eventual Consistency, in terms of money, business, user retention, and so on.
          - not just we need to be consistent period NO.
          - we could find places in which you could easily recover, or find workarounds.
          - ---
- ### Event-Sourcing (eventual consistency):
  - is the big brother of messages, has the idea of **the idea to store the events in a database and have those as the source of truth**.
  - Event sourcing stores state changes as a sequence of events rather than the current state. The current state is rebuilt by replaying these events, providing an audit trail, and enabling features like undo functionality.
  - #### Benefits and how it works:
    - **State Moves Via Events** --> less REST
    - **Lossless capture of History** wonderful for resilience because you can not lose anything in the past, anything that happens is stored as an event in an `Event Store(is a specialized storage system used to persist events. Instead of storing the current state of an entity directly)`.
    - **Time Travel Ability** by replaying the events, you can rebuild the state at any point in the past by replaying the event , amazing for reseliance you cannot loose stuff anymore.
    - **Briefly how it works**:
      - **1/Capturing Events**: Each change in state is recorded as an event. **2/Storing Events**: Events are saved in an append-only, time-ordered log. **3/Rebuilding State**: The current state is reconstructed by replaying these events.
      - **Characteristics**:
        - Append-only: Events are added, not modified or deleted.
        - Time-ordered: Events are stored in sequence.
        - Immutable: Events are permanent and unchangeable. 
  - ### Super Scalable ReadPart (CQRS):
  - if you want to go super scalable for the Reading Part, you can use CQRS and have multiple machines consuming from that single stream of events, and suddenly you can have 10 Machines serving the searches and the reads to meet a very high scalability need, eq like google search
  - ---
- ### Messaging Pitfall:
  - **Poison Pill messages**: send to **Dead-letter** or **retry queue**. (messages that blow up the listener and retried at infinitum on each restart of broker)
  - **Duplicated messages**: solution Must queue consumer be Idempotent or else a disaster and change in state occures.
  - **Out-of-Order messages**: solution use aggregator, time-based-messages, Smart-messages, stateful aggregator ... all sort of techniques
  - **Data Privacy**: solution claim check pattern (customer data stored in a queue for many days stuck)
  - **Event Versionning**: solution backward-forward compatibility
  - **broker down insufficient resources** solution Monitoring
  - **Misconfiguration of Infrastructure**:
  - 
---
---
## Supervision:
  - ### Health-Check:
    - its support important that our services exposes a health check endpoint like (spring boot actuator health).
    - that **tells some monitoring tools** what is the reason of the failure.
    - Ex:
      - service interfaces your ecosystem with a payment gateway, `if that payment gateway is down , your **health endpoint should tell that**` the system that i already depend on is down . so you can immediatly pinpoint on the root cause .
      - ---
  - ### Monitoring:
    - ---
  - ### Escalation:
    - why need it :
     - its really really tricky , when every time some works in a MS blows up, `to return an error to the caller, and maybe the caller can't solve that problem if its internal to that MS more than the logic itself` , it cant help you with anything , especially if using messages it can't know .
     - most Errors can't be fixed by the clients (unless we re talking about some payload or incorrect request)

     - instead of of `returning the error upstream(backwards in the flow)`  you could **escalate them** maybe call a supervisor(auto|manual) to reconcile that.
    - the `Flow of the process` is **orthogonal** to how the **error should be reported**.
      - ![escalation](images/escalation.jpg)
      - ---
  - ### Error-Handler:
    - #### People (embrace automation)
      - its always a people problem, no matter what problem seems to be at first.
      - if performance issue occures you re dead if no proper monitoring happens
      - **solution to get to those highly resilent system**:
        - you need **DevOps teams** tusi means you have your team responsible for the full lifecycle of your product from conception down to production monitoring and back.(perhaps with help of some pure blood OPS if needed)
        - **Own your Production** should know to :
          - see metrics && sets alarms (production metrics and all other metrics )
          - understand how to set a graph in grafana .
          - set an alarm
          - expose a metric
          - trace request across your systems & view the arrogated log fo what the request caused to happen
          - see time-spans of request to verify|optimise the **configuration/solution of latency control pattern**. tool eg zipkin.
