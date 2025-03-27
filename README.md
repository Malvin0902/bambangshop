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
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [x] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. **Do we need an interface (or trait in Rust) in BambangShop, or is a single Model struct enough?**

    In the Observer pattern, an interface (or trait) defines a contract all subscribers must follow. In **BambangShop**, we use a `Subscriber` struct without a trait, which is fine for now since all subscribers are the same. However, if we introduce different types of subscribers (e.g., email or webhook), a trait would be useful to define shared behavior, such as:

    ```Rust
    trait Subscriber {
        fn notify(&self, notification: &Notification);
    }
    ```

    For now, a trait isn’t strictly necessary but would improve flexibility later.

2. **Is using Vec sufficient, or is DashMap necessary?**

    Since `id` in **Program** and `url` in **Subscriber** must be unique, a key-value storage structure is ideal.

    - **Vec<Subscriber>**: Searching by `url` takes O(n) time, and removing a subscriber also requires iteration.
    - **DashMap<String, Subscriber>**: Provides constant-time (O(1)) lookups and ensures unique `url` keys.

    `DashMap` is the better choice for efficient lookups and management of subscribers.

3. **Do we need DashMap, or can we implement Singleton instead?**

    We currently use **DashMap** with `lazy_static!` for a global instance:

    ```Rust
    lazy_static! {
    pub static ref SUBSCRIBERS: DashMap<String, Subscriber> = DashMap::new();
    }
    ```

    **DashMap** ensures thread safety, unlike Singleton with `HashMap`, which would require `Mutex` or `RwLock` for locking overhead.

    **DashMap** provides both singleton behavior and thread safety, making it the best choice.

#### Reflection Publisher-2

1. **Why do we need to separate “Service” and “Repository” from the Model?**

    - **Separation of Concerns (SoC):**
      - **Repository:** Handles database access.
      - **Service:** Contains business logic.
      - **Model:** Represents data structure without business logic or database interaction.
      
    - **Scalability & Maintainability:** Separating layers makes refactoring and adding features easier without altering database logic.
    
    - **Testability:** The **Service** and **Repository** layers are easy to unit test without database dependencies.

    **BambangShop** follows a layered architecture with `NotificationService` and `SubscriberRepository` for modularity and maintainability.

2. **What happens if we only use the Model?**

    - **Tightly Coupled Components:** The Model would handle both business logic and database operations, making the code harder to maintain.
    
    - **Complex Interactions:** Reduces flexibility, e.g., `Notification` would interact directly with `Subscriber`, increasing dependencies.
    
    - **Difficult to Add Features:** Modifying the Model to add features like logging or validation may violate the **Single Responsibility Principle**.

    Without the **Service** and **Repository** layers, the code becomes more complex and harder to maintain.

3. **How does Postman help in testing our current work?**

    - **API Testing:** Allows testing `POST` for **Subscribe** and **Unsubscribe**.
    
    - **Postman Features:**
      - **Collections & Environments:** Groups requests and defines variables for different environments.
      - **Automated Testing:** Validates API responses automatically.
      - **Mock Servers:** Tests frontend without a backend.
      - **API Documentation:** Generates interactive API documentation for easy team understanding.

#### Reflection Publisher-3

1. **Which variation of the Observer Pattern do we use in this case?**

    **BambangShop** uses the **Push model**:
    - **Publisher** (NotificationService) actively sends notifications to subscribers.
    - When a product changes, `notify` sends notifications to all subscribers.
    - `Subscriber::update` sends an HTTP request to the subscriber's URL asynchronously.

2. **What if we used the Pull model instead?**

    **Advantages of Pull Model**:
    - Reduces load on the publisher.
    - Better handling of subscriber failures (subscribers can pull updates when they are online).
    - More control for subscribers.

    **Disadvantages of Pull Model**:
    - Higher complexity for subscribers.
    - Delayed notifications (not real-time).
    - Higher resource consumption on the system if many subscribers frequently poll.

    **Conclusion:** The **Push model** is better for **BambangShop** because it requires real-time notifications.

3. **What happens if we don’t use multi-threading in the notification process?**

    **Without multi-threading**:
    - Notifications would be slower and processed sequentially.
    - API response time would increase since all notifications must be sent before returning a response.
    - Failure in one notification blocks others from being sent.

    **With multi-threading**, notifications are sent efficiently, allowing the system to be faster and more scalable.