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
-   [✓] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [✓] Commit: `Create Subscriber model struct.`
    -   [✓] Commit: `Create Notification model struct.`
    -   [✓] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [✓] Commit: `Implement add function in Subscriber repository.`
    -   [✓] Commit: `Implement list_all function in Subscriber repository.`
    -   [✓] Commit: `Implement delete function in Subscriber repository.`
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

1. In the Observer pattern as explained in Head First Design Patterns, the Observer (Subscriber) is typically defined as an interface. In the BambangShop case, we could use a trait in Rust to define the behavior expected of subscribers, particularly an `update` method that gets called when notifications need to be sent. However, in this specific implementation, a single Model struct is sufficient, because
   - The behavior of all subscribers is uniform. They all receive HTTP POST requests
   - We are using a concrete implementation rather than multiple subscriber types
   - The notification sending logic can be implemented directly in the Subscriber model
   - Rust&apos;s trait system would be valuable if we had multiple types of subscribers with different notification handling mechanisms

2. Using DashMap rather than Vec (list) is necessary for this case, because
   - We need to efficiently lookup subscribers by their unique identifiers (url)
   - DashMap provides O(1) lookup operations compared to O(n) for Vec
   - When adding or removing subscribers, we need to check for duplicates quickly
   - The repository pattern benefits from having fast key-based access
   - The implementation needs to maintain uniqueness guarantees that are more efficiently handled with maps

3. The use of DashMap with lazy_static for the SUBSCRIBERS variable is a good approach, because
   - It combines aspects of the Singleton pattern (ensuring a single instance) with thread safety
   - DashMap specifically provides interior mutability that is thread-safe, which is necessary in a web server context where multiple threads access the same data
   - While we could implement a custom Singleton pattern, DashMap already provides the thread-safety guarantees we need
   - Rust&apos;s ownership system and compiler constraints lead us to solutions like DashMap that handle concurrency concerns at compile time
   - The lazy_static macro ensures the singleton is initialized only when first accessed

#### Reflection Publisher-2

#### Reflection Publisher-3
