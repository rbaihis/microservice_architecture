# In-Depth Talk When Building Microservices and distributed Systems :
---

## Distributed-System Defenition
  * A distributed system is one in which the failure of a computer you didn't even know existed, can render your own computer unusable.

## If your system can't withstand production you're doomed:
  * if your business idea can't be mapped into a system available at least 99% then the system we built is useless . customers will run away.
  * $availibility=uptime/total_time=bar{meanTimeToFailure}/(bar{meanTimeToFailure})+bar{meanTimeToRecovery}$
