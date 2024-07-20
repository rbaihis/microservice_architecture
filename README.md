# Microservice Architecture
This is my documentation for how I understand microservice it's not a holy book so feel free to argue about and correct me.

## what is a microservice:
  - is a physically deployable piece of software.
  - microservice is independent, loosely coupled, and focused on single functionalities or business capabilities.

## Rule  of thumb:
  - It is fine to have a hybrid (mono-microservice) system, where some modules remain monolithic to minimize costs when they do not benefit significantly from being split into microservices.
  - Note:
    - Hybrid systems can be cost-effective, They should still be carefully evaluated for potential integration and complexity issues.
  - you can't excel in microservice architecture if you don't understand Domain driven architecture , DDD helps in identifying and defining bounded contexts, which are foundational for effective microservice decomposition.
  - Architecting GreenField project :
    - Since the system is expected to grow and evolve, starting with a monorepository can facilitate easier restructuring and rearchitecting without significant overhead.
    - why: because ur system will difenetly grow since u decided its a should be ms in the first place. and there is no clear view about the system since its still fresh. Using a <b>monorepositor</b>y can be beneficial. It simplifies the initial development and integration process. As the system grows and matures, having all the microservices in a single repository can make refactoring and architectural changes easier.

## Statements about ms not 100% correct:
  - **Each microservice should have its own database** :
    + ==> `Each bounded context should have its own database`, not necessarily each microservice.</b>
  - **Data Consistency matters 100%**:
    - No sometimes data consistency does not matter and better to avoid it to not over complex your microservices .
    - only use it when it matters most :
      - Ex (when it does not matter ): you are collecting data for analytics or forecasting loosing few data will not matters .
      - Ex (when it matters ): accounting, banking, finance, orders, payment , database recovery logging , etc ... 

## A bounded context is not equivalent to a microservice:
  - ## ## Review on Bounded Context: 
    - A bounded context in Domain-Driven Design (DDD) is a logical boundary, while a microservice is a physical deployment unit. Although they often align, they are not the same.
  - ## In many cases : 
    - one bounding context => implemented as exactly one microservice.
  - ## Splitting a bounding Context (common as well):
    - There are technical reasons to split a bounded context into multiple deployable units   microservices.
    - ex:
      - separating: online services from housekeeping jobs that are running periodically.
        
## Ask yourself :
  - ### Should that be a microservice:
    - **1- Multiple rates of change**:
        - Do parts of your system need to evolve at different speeds or in different directions? I(yes=> ms)
        - ex: (E-Commerce) :
          - cart and inventory are rarely touched, but we often like to improve our recommendation functionalities and optimize search functionalities.
    - **2- Independent lifecycles** :
      - Does the module should have an independent lifecycle, its own pipeline(ci/cd) from build to production ? ==> (yes ==> microservice) 
    - **3- Independent scalability** : 
      - if load or throughput characteristics of parts of the system are different ==> they may have different scaling requirements (avoid scaling the whole monolithic by converting them to microservices)
    - **4- Isolated failure** :
      -  we want to insulate our app from a particular type of failure.
      -  Ex:
        - Your app depends on an unreliable third-party service with poor performance, it is better to have a microservice that directly interacts with that service fails rather than the whole application.
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
        - keeping a reliable legacy system wile adding on the new microservice with a better technology.
        - using Python for IA and data science tasks, java for critical apps, and C++/Rust for low-level solutions.
        - if you have a team of 6-engineers, it is better to stick to one technology.
        - use this wisely to avoid reverse effect.
## What set of problems should we consider microservices :
  - #### Avoid developers-production toxic relationship :
    - no more projects ==> talking about products you build your product you deploy it.
  - #### Enterprise-service-bus only used as dump pipes no logic:
    - its safe to ignore and convert you service-oriented to microservices
  - #### microservices avoid service buses:
    - Modern microservice architectures generally recommend avoiding ESBs because they can introduce tight coupling and become bottlenecks. Smart endpoints and dumb pipes (simple communication mechanisms like HTTP or messaging) are preferred.
  
## When to avoid considering microservices :
  - #### Enterprise service-bus uses logic:
    - requires consideration (tight coupling issue)
    - why: information that is included in the service-bus is really part of the bounded context and you re violating that bounded context phylosofy of micro services by trying to take some of that contexts , externalizing it in an other moving part when we re trying to break the coupling between the moving part in the architecture here to create the shared "nothing kind of architecture".



## Data consistency is difficult in Ms-Architecture :
  - achieving data consistency in micro service is quite complex, it requires a careful though 
    
