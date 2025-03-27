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

##### 1. Do we still need an interface (or trait in Rust) in this BambangShop case, or is a single Model struct enough?  
In the **Observer Design Pattern**, the **Subscriber** (Observer) is typically defined as an interface (or trait in Rust) to allow multiple different implementations. This makes sense in cases where we expect various types of subscribers that might have different behaviors.  

However, in the **BambangShop** case, we currently have a simple notification system where all subscribers are treated the same—each has a URL and a name. Since all subscribers follow the same structure, we do **not** need a separate trait; a single `Subscriber` struct is sufficient. If in the future we have different types of subscribers with unique behaviors, introducing a trait would be beneficial.  

##### 2. Is using `Vec` (list) sufficient, or is `DashMap` (map/dictionary) necessary for this case?  
A **Vec** (list) is useful when we have a small, fixed-size collection of items and mostly perform sequential access. However, in this case, `url` in `Subscriber` must be **unique**. If we use a **Vec**, checking for uniqueness requires iterating over all elements, making insertion and deletion inefficient (O(n) complexity).  

By using a **DashMap** (which is a thread-safe HashMap), we ensure:  
- **Efficient lookups** (O(1) complexity), as we can check if a URL exists instantly.  
- **Thread safety**, allowing multiple threads to modify subscribers without explicit locking.  
- **Concurrency support**, which is useful for a web application handling multiple users simultaneously.  

Thus, **DashMap is necessary** for ensuring both uniqueness and efficient concurrent access.  

##### 3. Do we still need DashMap, or can we implement the Singleton pattern instead?  
The **Singleton Pattern** ensures that only one instance of a variable exists globally. We could implement **SUBSCRIBERS** as a **Singleton using Rust’s `lazy_static!` macro** or the **OnceCell** crate.  

However, the main issue is **thread safety**. A standard `HashMap` inside a Singleton **would not be thread-safe**, meaning we would need additional synchronization mechanisms like `RwLock<HashMap<...>>` or `Mutex<HashMap<...>>`. These would introduce locking overhead and potential deadlocks in concurrent scenarios.  

**DashMap is already a concurrent-safe HashMap, removing the need for manual locks.** It provides:  
- **Fine-grained locking**, reducing contention between threads.  
- **Better performance** compared to using a `Mutex<HashMap<...>>` manually.  
- **Built-in support for multi-threaded operations.**  

Thus, while we **could** implement a Singleton manually, **DashMap already provides all the benefits of Singleton while also being optimized for concurrency**. Keeping DashMap is the better design choice.  

#### Reflection Publisher-2

##### 1. Why do we separate “Service” and “Repository” from a Model?  
In traditional **Model-View-Controller (MVC)**, the **Model** is responsible for both business logic and data storage. However, following **Separation of Concerns (SoC)** and **Single Responsibility Principle (SRP)**, it is beneficial to separate the **Service** and **Repository** layers:  

- **Service Layer**: Handles business logic, ensuring that rules, validations, and processing are centralized rather than being scattered in the Model.  
- **Repository Layer**: Abstracts data persistence and retrieval, enabling easier database switching and reducing code duplication.  

This separation provides better maintainability, testability, and modularity, making our codebase **scalable and easier to extend**.  

##### 2. What happens if we only use the Model?  
If we **only use the Model**, several problems arise:  

- **Tightly Coupled Code**: The **Program**, **Subscriber**, and **Notification** models would need to communicate directly, leading to **interdependent logic** that is hard to maintain.  
- **Increased Complexity**: Without a **Service** layer, each Model would need to handle its own business rules, making modifications or adding new features cumbersome.  
- **Difficult Unit Testing**: Business logic would be mixed with data access, making it difficult to write **isolated unit tests**.  

For example, if the **Notification** model directly manages subscriptions and data persistence, updating how notifications work would require modifying multiple parts of the Model. This creates a **ripple effect** that increases the risk of **breaking existing functionality**.  

##### 3. How does Postman help in testing our work?  
Postman is an essential tool for testing **RESTful APIs**. It allows us to **send HTTP requests**, verify responses, and automate tests. Some useful features:  

- **Request Collections**: Organizing API endpoints for structured testing.  
- **Environment Variables**: Managing different configurations (e.g., dev, staging, production).  
- **Automated Testing with Scripts**: Writing test assertions to verify API responses.  
- **Mock Servers**: Simulating API responses without a live backend.  

For our **BambangShop** project, Postman helps us quickly test **notification subscriptions**, **API routes**, and **error handling** before integrating with the frontend. This reduces debugging time and ensures a smoother development process.  

In future projects, Postman will be beneficial for **CI/CD pipelines**, **performance testing**, and **collaboration in API development**.  

#### Reflection Publisher-3

##### 1. Observer Pattern Variation Used  
In this tutorial, we use the **Push model** of the Observer Pattern. The publisher (our system) actively **sends notifications** to subscribers when an event occurs, rather than waiting for subscribers to request updates.  

##### 2. Advantages and Disadvantages of Using the Pull Model  
If we were to use the **Pull model**, where subscribers fetch data from the publisher instead of receiving updates automatically, the system would behave differently:  

- **Advantages of Pull Model:**  
  - Reduces unnecessary updates if subscribers only need data occasionally.  
  - Subscribers have more control over when and how they retrieve information.  

- **Disadvantages of Pull Model:**  
  - Increases complexity because subscribers must regularly check for updates.  
  - Can introduce delays if a subscriber does not check frequently enough.  
  - Higher network/resource usage if polling happens too often.  

##### 3. Impact of Removing Multi-threading in Notifications  
If we **do not use multi-threading** in the notification process, several issues may arise:  

- **Slower notifications:** Since each subscriber would be notified sequentially, a slow network or unresponsive subscriber could delay notifications to others.  
- **Blocking operations:** The main application could become unresponsive while waiting for notifications to complete.  
- **Reduced scalability:** Handling multiple subscribers would become inefficient as the number of subscribers grows.  

By using **multi-threading**, notifications can be sent concurrently, improving performance and system responsiveness.  
