# In-Depth Talk When Building Microservices and distributed Systems :
---

## Distributed-System Defenition
  * A distributed system is one in which the failure of a computer you didn't even know existed, can render your own computer unusable.

## If your system can't withstand production you're doomed:
  * if your business idea can't be mapped into a system available at least 99% then the system we built is useless. customers will run away.
  * $availability = upTime/totalTime =  \frac{\bar{\text{meanTimeToFailure}}}{\bar{\text{meanTimeToFailure}} + \bar{\text{meanTimeToRecovery}}}$

##  Key Things to Do to Mitigate Failure:
  * write more tests: unit-tests, end-to-end, as well as (load-test, spike, resilience, [ToxyProxy_tool_to_delay_traffic_between_services_for_testing](https://github.com/Shopify/toxiproxy/blob/main/README.md) )
  * try to recover from failure as fast as you can, we want to have a detection mechanism and ways we can backup as soon as possible.
---

## Resilience patterns:
 * **What is a Resilient System**: is the ability of a system to handle **unexpected** situations `without users noticing`(best-case), or with a `graceful degradation` of service.
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
     - sol 3: **Case: too frequent Errors**: send an error event to `supervise for automatic Recovery((saga pattern)[])`.
  - **Problem6**: 
  - **Problem7**: 
