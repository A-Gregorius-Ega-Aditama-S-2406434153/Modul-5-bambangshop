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
In this BambangShop case, I do not think we need a separate `Subscriber` trait yet. In the classic Observer pattern, an interface is useful because the publisher may talk to many different observer implementations through one common contract. Here, every subscriber is represented by the same data shape, namely a `Subscriber` struct with `url` and `name`, and the interaction model is also uniform because notifications are sent to an HTTP endpoint. Since there is currently only one concrete subscriber representation and no alternative observer behavior being swapped in polymorphically, a single model struct is enough. A trait would become useful only if we later needed several subscriber types with different update behaviors or transport mechanisms.

For uniqueness, using a plain `Vec` is possible only if we are willing to manually scan the list every time we add, delete, or validate a subscriber. That would make duplicate prevention and lookup depend on linear search. Because `url` in `Subscriber` is intended to be unique for each `product_type`, the current `DashMap<String, Subscriber>` is a better fit: the unique key is explicit, insertion naturally overwrites or rejects by key policy, and deletion by `url` is straightforward. The outer map grouped by `product_type` plus the inner map keyed by `url` matches the domain model more directly than a list.

For the static `SUBSCRIBERS` variable, `DashMap` and Singleton solve different problems, so Singleton is not a replacement for `DashMap`. A Singleton pattern only ensures there is one shared instance of some repository or store. It does not by itself make concurrent reads and writes safe. In Rust web applications, requests may be handled concurrently, so shared mutable state still needs synchronization. In this repository, `lazy_static!` already gives us one global shared store, while `DashMap` provides the thread-safe concurrent access we need. So the better conclusion is: if we keep global shared subscriber storage, we still need a thread-safe container such as `DashMap` or another synchronized structure, even if we describe the repository itself conceptually as a singleton.

#### Reflection Publisher-2
Separating `Service` and `Repository` from `Model` makes the code easier to maintain because each layer has one clear responsibility. The model should mainly describe the data shape, such as `Product`, `Subscriber`, and `Notification`. The repository should focus on persistence concerns like storing, retrieving, and deleting data from `SUBSCRIBERS` or `PRODUCTS`. The service should contain business rules, orchestration, and validation, such as normalizing `product_type` to uppercase and deciding what to do when a subscriber is not found. Even though MVC often groups storage and business logic under the broad idea of "Model," separating them in implementation follows single responsibility and makes the code easier to test, extend, and reason about.

If we only used the Model layer, each model would start carrying too many responsibilities at once. For example, `Subscriber` would not only represent subscriber data, but might also need to know how to store itself, remove itself from shared state, validate uniqueness, normalize product types, and maybe later send notification requests. `Notification` might also need to coordinate with subscribers and products directly, while `Product` could become responsible for publishing events and finding observers. That kind of design would tightly couple the models together and make each change more dangerous, because modifying one model would likely affect the persistence and business behavior of the others. Splitting the responsibilities across model, repository, and service keeps those interactions more controlled and prevents the models from turning into large "god objects."

I explored Postman more while working on this project, and it is very helpful for testing REST APIs quickly without writing custom client code first. For the current work, Postman helps me verify endpoints like subscribe and unsubscribe by letting me set the HTTP method, route, query parameters, and JSON body, then inspect the returned status and payload immediately. I am also interested in Postman collections, environment variables, and automated tests inside requests, because those features would help a group project keep API checks consistent across team members. For future software engineering projects, I think Postman will stay useful both as a manual testing tool during development and as lightweight documentation for how an API should be used.

#### Reflection Publisher-3
