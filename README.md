# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
# 1. Question 1
In the current implementation, the subscriber code is only defined as a struct with 2 fields (url and name), there's no methods or behaviour defined on it. So, in my opinion Subscriber acts purely as a data container not as an active observer that defines its own notification handling. For now, a single model struct is enough because there's no behaviour to abstract (the struct itself doesn't define how notifications are delivered). 

However if in the future we have different types of subscribers that need to handle the notifications differently (example some via HTTP, others via email or other protocols) then using a trait would be the right approach. The trait act as an interface that defines a contract (notify() method) that all subscriber types must implement making the system more aligned with the observer pattern's intent. 

# 2. Question 2
Vec is not sufficient because we need a data structure that can guarantee that the url in subscriber and id in Program must be unique,  vec doesn't inherently enforce those uniqueness. This is similar to a manual attendance  sheet where the same person can sign in multiple times  without the system noticing.

Dashmap is the right choice for this case because it use these unique identifiers (id/url) as keys which inherently enforces uniqueness by using this, duplicate ids or urls simply cannot exist anymore just like a fingerprint attendance system that automatically rejects 
duplicate entries.


# 3. Question 3
We still need a dashmap event though we've implement the singleton pattern because they solve different problems. Singleton only ensures that there's only one instance of SUBSCRIBERS exists globally but it doesn't handle thread safety.

Dashmap is spesifically designed as a thread safe hashmap, allowing multiple threads to read/write simultaneously without data races. So in conclusion we still need dashmap to ensure safe concurrent access to subscriber even though we've used singleton. 

#### Reflection Publisher-2
# 1. Question 1

As the application grows, putting everything in the Model violates the Single Responsibility Principle (SRP) where each component should only have one reason to change.

We need to seperate service and repository from model because if we put everything into model, any change in storage structure would also affect the business logic and vise versa, it will make the code harder to mantain and test. By separating service and repository from model, each layer has clear responsibility, the model defines the data structure, repository handles data access and storage, service handles business logic. 


# 2. Question 2
If we only use the Model, each model (Program, Subscriber, Notification) would need to directly handle their own data storage and business logic. This means they would be tightly coupled to each other for example, Program model would need to directly access Subscriber model to notify subscribers, and Subscriber model would need to know how to deliver Notification. 

This creates a web of dependencies between models, where any change in one model would likely break the others. The code complexity would increase significantly as each 
model grows bloated with responsibilities it shouldn't have, making the codebase much harder to maintain, test, and scale.

# 3. Question 3
Yes, i've used postman for the platform based programming course previously. The postman is very helpful to test the API endpoints. There's a feature called collection that help me to organize and save all endpoints in one place, making it easy to test many endpoints without rewriting the requests everytime. I also find that postman's ability to set request body in JSON format makes it straightworfawrd to test different scenarios. 

Postman also supports environment variables which allows me to switch between different base URLS without manually editing each reqyest. The automated testing feature using test scripts is also useful to automatically validate responses, such as checking status codes or response body values, making the testing process more efficient and reliable. 

#### Reflection Publisher-3
# 1. Question 1
In this tutorial we use push model, NotificationService actively send the data directly to every subscriber with an update() method. Subscriber don't need to ask and request data to publisher, they simply just receive and process data that has been pushed to them. Subscriber must understend the data structure that the publisher sends. 

# 2. Question 2
Advantages :
- Data efficiency
If a subscriber only requires spesific information, they are not forced to receive a large/complete object payload. In a pull model, the subscriber can selectively fetch only data field they actually need, which it can save memory and reduce network traffic. 

- Subscriber autonomy
Subscriber can decide exactly when they're ready to process or retreive data. 

- Reduced coupling
The subscriber doesn't need to be perfectly synced with the publisher's data
structure. If the publisher add new fields to the main object, the subscriber's logic wont break. That's because they only pull the spesific data they we're already programmed to handle. 

Disadvantages:
- Increased Latency & Overhead
The pull model requires two way communication. First the publisher sends a notification and then the subscriber must send a seperate request back to the publisher to get the data. This increase the time it takes to complete the entire process. 

- Increased Architectural Complexity
The publisher must implement and mantain additional API endpoints so that subscribers can reach back and pull the data. This make the system more complex to build compared to a simple one way push

# 3. Question 3
- The program will send notifications to each subscriber one by one in sequential order. If one subsscriber has a slow response time, the entire process will be blocked / the next subsceiber in the list must wait till the previous one is finished before it can receive its notification

- Actions like creating, deleting, or publishing a product will feel very slow. The API response will only be sent back to the user after every single subscriber has been successfuly notified.

- If the update() method on subscriber encounters a long delay, it could cause the entire NotificationService to hang. 

- If the subscriber list grows large, the total notification time scales linearly, making the system increasingly unsresponsive under heavy load
