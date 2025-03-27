# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

1. **Why is `RwLock<>` necessary instead of `Mutex<>`?**

    In our receiver app, we use `RwLock<Vec<Notification>>` to synchronize access to the notification list. This is necessary because multiple threads can read notifications at the same time, but only one thread should be allowed to write at a time. There are multiple reason why using `RwLock<>` is better than the latter. Some of them are:

    -   Multiple readers are allowed (multiple threads can read notifications concurrently).
    -   Efficient for read-heavy operations, as reading is not blocked unless there is a writer.
    -   Prevents race conditions, ensuring data consistency.

    With that being said, there are a couple of reasons on why not to choose `Mutex<>`, some examples are:

    -   `Mutex<>` only allows one thread to access the data at a time, even for reading.
    -   Since notifications are mostly read (e.g., when listing all notifications), using `Mutex<>` would block all other reads whenever a thread is writing.
    -   This would reduce performance, especially if there are many subscribers requesting notifications at the same time.

    We use `RwLock<>` because it provides better performance for a read-heavy use case. If we used `Mutex<>`, it would block readers unnecessarily, leading to slower performance.

2. **Why does Rust not allow mutation of static variables like Java?**

    In Java, we can mutate static variables using static functions, but Rust does not allow direct mutation of `static` variables because of its safety guarantees. There's a couple of reason why Rust restricts mutable `static` variable, some of them are but not limited to:

    -   Rust enforces thread safety at compile time.
    -   If `static mut` was allowed, it could cause data races in multi-threaded applications.
    -   Java uses a Garbage Collector (GC) to manage memory, whereas Rust strictly enforces ownership and borrowing rules to ensure safe memory access.

    But if Rust does not allow mutation of static variables, how come it solve the problems that required mutation? Now instrad of allowing direct mutation of `static` variables, Rust provides safe alternatives such as:

    a. Using `lazy_static!` (like in our case)

    -   We wrap `Vec<Notification>` inside an `RwLock<>` to ensure safe concurrent access.
    -   This prevents data races while still allowing global access.

    b. Using `DashMap` (for concurrent data structures)

    -   Unlike `Vec`, `DashMap` is designed for concurrent reads and writes, allowing safe updates to shared data.

    To summarize, Rust prioritizes safety over convenience. Unlike Java, where static variables can be mutated freely, Rust requires explicit synchronization (`RwLock<>`, `Mutex<>`, or `DashMap`) to prevent data races. Using `lazy_static!` allows us to create a safe, globally accessible, mutable data structure.


#### Reflection Subscriber-2

1. **Have you explored things outside of the steps in the tutorial (e.g., src/lib.rs)?**

    **Yes**, I have explroed additional parts of the code, such as `src/lib.rs` and `src/main.rs`. Here are some thing that I have learned:

    a. From `lib.rs`

    - Global Configuration Handling with `lazy_static!`

        -   `APP_CONFIG`: Stores configuration values such as `instance_root_url`, `publisher_root_url`, and `instance_name`.
        -   `REQWEST_CLIENT`: A pre-initialized HTTP client for making network requests efficiently.

    - Configuration from Environment Variables

        -   Using `dotenvy::dotenv()` and `Figment`, the app can load configurations dynamically, making it easier to deploy across different environments. Changes from `.env` such as `APP_INSTANCE_NAME=Andriyo Averill` and changing the `ROCKET_PORT` to be 8001, 8002, 8003 respectively.

    - Error Handling (`compose_error_response`)

        -   The system follows a structured error response format using `Custom<Json<ErrorResponse>>`, ensuring consistency in API error messages.

    b. From `main.rs`

    - Application Bootstrapping

        -   The `#[launch]` function initializes the Rocket web framework and loads environment variables using `dotenv().ok()`.
        -   Calls `rocket::build()` to create a new instance of the Rocket web server.

    - Dependency Injection (`manage()`)

        -   The `reqwest::Client` is pre-initialized and managed within Rocket, ensuring efficient HTTP requests across the application.

    - Route Registration (`attach(route_stage())`)

        -   `route_stage()` from `controller::mod.rs` mounts all API routes in a modular way.

    Now, if I hadn’t explored these parts, I might not have fully understood how the application boots up, manages dependencies, and registers routes dynamically.

2. **How does the Observer pattern help with adding subscribers?**

    The Observer pattern is useful because it decouples publishers from subscribers, making it easy to add new instances dynamically.

    **Benefits in the Notification System**

    - Easily Add More Subscribers

        A new `Receiver` instance simply calls `subscribe(product_type)`, and the publisher registers it without modifying existing code. The NotificationService abstracts subscription logic, so adding new subscribers doesn’t require changes in the publisher.

    - Efficient Notification Handling

        Each `Receiver` instance listens for updates independently. The publisher doesn’t need to know how many subscribers exist since it just notifies all registered ones.

    **Spawning multiple `Main` Instances**

    -   Spawning multiple Publisher (Main) 

        It is more complex since if multiple publishers exist, they must coordinate to ensure notifications are sent correctly. But there are some possible solutions if we're still trying to do so, such as Centralized Database. In this case, Publishers share a common database of subscribers.

    Thus, while adding subscribers is easy, adding publishers requires additional synchronization mechanisms.

3. **Have you tested with Postman or written custom tests?**

    **Yes**, I have used Postman for testing and explored additional features. Since we have to have three different instances of Receiver app and one instance of Main app running simulatenously, if we have each Postman test and changing the port from 8001, to 8002, or even to 8003, it would take so much time to do so. Instead, we can create custom API Tests in Postman and classifies them as a collection. On the group project, for now, since we haven't implemented the path or even succesfully built it, I can't really say much. But, we (at least my group and I) are going to use Postman as our API service to test out some path.