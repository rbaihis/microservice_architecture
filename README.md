# Microservice Architecture
This is my documentation for how I understand microservice it's not a holy book so feel free to argue about and correct me.

## what is a microservice:
  - is a physically deployable piece of software.

## Rule  of thumb:
  - It is fine to have a hybrid (mono-microservice) system, where the monolithic modules do not meet the need for microservice architecting and are better kept monolithic for minimizing costs.

## A bounded context is not equivalent to a microservice:
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
          - Microservices can act as <bold>a layer of indirection<bold> 
    - **6-**
## What set of problems should we consider microservices :
  #### Avoid developers-production toxic relationship :
    no more projects ==> talking about products you build your product you deploy it.
  #### Enterprise-service-bus only used as dump pipes no logic:
    its safe to ignore and convert you service-oriented to microservices
  #### Enterprise service-bus uses logic:
    requires consideration (tight coupling issue)
    - why: information that is included in the service-bus is really part of the bounded context and you re violating that bounded context phylosofy of micro services by trying to take some of that contexts , externalizing it in an other moving part when we re trying to break the coupling between the moving part in the architecture here to create the shared "nothing kind of architecture"
  #### microservices avoid service buses:
    instead: smart endpoints,Dump pipes => commonly (HTTP/Messaging)
  
## When to avoid considering microservices :
  #### Enterprise service-bus uses logic:
    requires consideration (tight coupling issue)
    - why: information that is included in the service-bus is really part of the bounded context and you re violating that bounded context phylosofy of micro services by trying to take some of that contexts , externalizing it in an other moving part when we re trying to break the coupling between the moving part in the architecture here to create the shared "nothing kind of architecture".
    
