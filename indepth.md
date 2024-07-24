# Reseliance PatternsMust Know Building Microservices :
</br>

## Distributed-System Defenition
  * A distributed system is one in which the failure of a computer you didn't even know existed, can render your own computer unusable.

## If your system can't withstand production you're doomed:
  * if your business idea can't be mapped into a system available at least 99% then the system we built is useless. customers will run away.
  * $availability = upTime/totalTime =  \frac{\bar{\text{meanTimeToFailure}}}{\bar{\text{meanTimeToFailure}} + \bar{\text{meanTimeToRecovery}}}$

##  Key Things to Do to Mitigate Failure:
  * write more tests: unit-tests, end-to-end, as well as (load-test, spike, resilience, [ToxyProxy_tool_to_delay_traffic_between_services_for_testing](https://github.com/Shopify/toxiproxy/blob/main/README.md) )
  * try to recover from failure as fast as you can, we want to have a detection mechanism and ways we can backup as soon as possible.
___
___


## Resilience patterns:
 * **What is a Resilient System**:(trying to do your best not to show the failure) is the ability of a system to handle **unexpected** situations `without users noticing`(best-case), or with a `graceful degradation` of service.
 ### Example of Graceful degradation (Read-problems) :
  - **Problem1**: trying to retrieve data from a system that is down ==> you could serve an older value and hope nothing can go wrong importance of caching.
  - **Problem2**: can't access user recommendation ==> return lower quality as recommendation per country, city, or gender better than nothing.
  - **Problem3**: search optimiser not working ==> call a slower system, for example, search in sqlDb when ElasticSearch is down better than nothing.
  - **Problem4**: can't send result immediately ==> notify by we will send you the result later in an email when we're done.
### Example of Graceful degradation (Write-problems) :
 - **Problem5**: command to change data.
 - sol 1: **outbox table pattern**: insert it in DB of the caller service the request + schedule a persistant retly until successful
 - sol 2: **send it as message**: instead of HTTP use asynchronous MQ approach and the system will handle it when available (handle end-user retry gracefully)
 - sol 3: **Log an ERROR**: at least log an error, that raises the alarm calling for manual intervention.
     - sol 3: **Case: too frequent Errors**: send an error event to `supervisor for automatic Recovery([saga pattern]())`.
## Patterns we will study :
![patterns we will study](images/patterns.png)

***
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
  - ![draw_boundaries](images/boundaries.png) <br>
  - **1st** we need to identify the pieces in our architecture that should still work, even if other parts on our system are down .
  - **2nd** you should draw these lines and say whatever in green should not depend on stuff on blue or yellow.
  - **3rd** if the blue has to work reasanbly without the green ones , you would have to `cache(replicate) some data` of the green one to survive them being down (`use these boundaries as boundaries of redundancy`).
  - **4th** use the bundaries as **units of scale** enough to scale up that bubble and not every single service.
     

***
