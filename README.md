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

## Mandatory Checklists (Publisher)
-   [✓] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [✓] Commit: `Create Subscriber model struct.`
    -   [✓] Commit: `Create Notification model struct.`
    -   [✓] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [✓] Commit: `Implement add function in Subscriber repository.`
    -   [✓] Commit: `Implement list_all function in Subscriber repository.`
    -   [✓] Commit: `Implement delete function in Subscriber repository.`
    -   [✓] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [✓] Commit: `Create Notification service struct skeleton.`
    -   [✓] Commit: `Implement subscribe function in Notification service.`
    -   [✓] Commit: `Implement subscribe function in Notification controller.`
    -   [✓] Commit: `Implement unsubscribe function in Notification service.`
    -   [✓] Commit: `Implement unsubscribe function in Notification controller.`
    -   [✓] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
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

1. Separation of concerns is a fundamental design principle that guides us to divide responsibilities, allowing each component to focus on specific tasks and making the system more modular and maintainable. The Single Responsibility Principle says that each class should have only one reason to change. Separating data access (Repository) from business logic (Service) directly follows this principle. This separation enhances testability, as we can write unit tests for business logic without database connections. It also provides flexibility, allowing us to change database implementations without touching the business logic, or update business rules without modifying data access code. The architecture becomes more scalable as separated components can be optimized independently, and repository methods can be reused across different services, reducing code duplication.

2. Without separate Service and Repository components, each model would need to handle data storage, business rules, and interactions with other models, resulting in large, complex classes with tight coupling. Models would directly depend on each other, making changes difficult as modifications to one model could affect others. Maintainability would suffer significantly as business rule changes would require modifying code mixed with data access logic, increasing the risk of bugs. Testing would become challenging with intertwined data storage and business logic. In our specific web, the model would need to incorporate notification logic and subscriber management. the Subscriber model would handle its own persistence and notification delivery, and the Notification model would contain sending logic plus creation rules. Any protocol change would require modifications in multiple places throughout the codebase.

3. Postman has been incredibly helpful for testing API endpoints through its intuitive request building interface that supports various HTTP methods. The Collections feature allows organizing related requests for subscriber management, product operations, and notifications. Environment Variables enable testing against different environments, while Request History provides access to previously sent requests during debugging. Automated Testing capabilities let me create test scripts to validate responses, ensuring endpoints work correctly. The Documentation generator creates API documentation from collections, facilitating team collaboration. Mock Servers allow frontend testing without waiting for backend completion. Collaboration Features support sharing collections with team members, while Response Visualization makes it easier to understand API behavior through formatted JSON responses. Pre-request Scripts are valuable for setting up prerequisites before each request, particularly for authentication or data preparation scenarios.

#### Reflection Publisher-3

1. In this tutorial case, we are using the Push model variation of the Observer Pattern. This is shown from our implementation where the publisher actively pushes notifications to all subscribers whenever a relevant event occurs, such as product creation, promotion, or deletion. When these events happen, our notification service directly calls the update method on each subscriber, sending HTTP POST requests with the notification details to their registered endpoints. The subscribers don't need to periodically check or request updates from the publisher, instead they passively wait to receive notifications when something changes.

2. If we had used the Pull model instead, subscribers would need to periodically request updates from our system to check if any relevant events had occurred. The main advantage would be reduced server load during high-volume events, as our system wouldn't need to immediately process and send many outgoing requests. Additionally, subscribers would have more control over when they receive updates and could fetch them according to their own schedule or bandwidth availability. However, the disadvantages would be significant. Notifications wouldn't be real-time, leading to potential delays in subscribers receiving important updates, we would need to implement session management to track changes between subscriber pulls, subscribers would waste resources making requests when no changes have occurred, and the system would become more complex overall, requiring additional endpoints for subscribers to query and mechanisms to determine what updates to return based on the subscriber's last check time. For a notification system like ours where timely updates are important, the Push model is more suitable.

3. Without multi-threading in the notification process, our program would experience significant performance issues. Since sending HTTP notifications is an I/O-bound operation that takes time to complete due to network latency and waiting for responses, the entire application would block during the notification sending process. This means that when a product action triggers notifications to multiple subscribers, each notification would be sent sequentially, and the system would be unable to handle any other requests until all notifications are finished sending. This would create a poor user experience where actions like creating or deleting products would appear to freeze or time out, especially when there are many subscribers or when network conditions are slow. In worst-case scenarios, notification timeouts could crash the application or cause incomplete operations. Multi-threading allows these notifications to be sent in parallel in the background, keeping the main application responsive and significantly improving performance and user experience.
