# SafeTix: High-Concurrency Booking Engine

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-0.95%2B-009688?style=for-the-badge&logo=fastapi)
![SQLite](https://img.shields.io/badge/SQLite-ACID%20Compliant-003B57?style=for-the-badge&logo=sqlite)
![Status](https://img.shields.io/badge/Build-Passing-success?style=for-the-badge)
## 📊 Performance Validation

I simulated a high-load environment with 50 concurrent users. The results below demonstrate the necessity of transaction locking in distributed systems.

![Stress Test Results](images_(proof_of_work)/tester.png)

> **Observation:** In Unsafe Mode, the race condition led to inconsistent state (Traffic too low error or Overselling). In Safe Mode, the `BEGIN IMMEDIATE` transaction ensured 100% accuracy.

## 🛠 API Reference

The API is fully documented and testable via the Swagger UI.

![Swagger UI](images_(proof_of_work)/Swagger_UI.png)
## 📋 Executive Summary
**SafeTix** is a high-performance backend REST API designed to simulate and resolve **Race Conditions** in high-traffic e-commerce environments. 

In real-world distributed systems (such as Flash Sales or Concert Bookings), concurrent requests often lead to **Inventory Overselling** due to non-atomic database operations. This project demonstrates a production-grade solution using **Pessimistic Locking** and **ACID Transactions** to ensure data integrity under heavy load.

The system features a comparative architecture, allowing developers to test "Unsafe" (naive) vs. "Safe" (engineered) implementations against a custom-built multi-threaded stress testing suite.

---

## ⚙️ System Architecture & Engineering Principles

### The Problem: The "Read-Modify-Write" Race Condition
In a naive implementation, purchasing a ticket involves three steps:
1.  **READ:** Check if `tickets > 0`.
2.  **MODIFY:** Decrement `tickets` by 1.
3.  **WRITE:** Save the new value to the database.

Under high concurrency (e.g., 50 requests/second), multiple threads pass Step 1 before any thread reaches Step 3. This results in the system selling inventory that does not exist.

### The Solution: Database Transaction Isolation
SafeTix solves this by implementing **Serializability** at the database level.
* **Mechanism:** SQLite `BEGIN IMMEDIATE` Transaction.
* **Behavior:** When a "Buy" request initiates, the database row is locked. Subsequent requests must wait in a queue until the transaction commits or rolls back.
* **Result:** Operations become atomic. The database state is guaranteed to be consistent, preventing overselling 100% of the time.

---

## 🛠️ Tech Stack

* **Backend Framework:** [FastAPI](https://fastapi.tiangolo.com/) - Chosen for its asynchronous capabilities and automatic OpenAPI documentation.
* **Database:** SQLite - Configured with **Write-Ahead Logging (WAL)** to handle concurrent reads/writes efficiently.
* **Validation:** [Pydantic](https://docs.pydantic.dev/) - Enforces strict data schemas for API requests.
* **Testing:** Python `threading` & `requests` - Custom-built script to simulate distributed client load.
* **Server:** Uvicorn - ASGI server implementation.

---

## 📂 Project Structure
The project follows a modular "Separation of Concerns" design pattern suitable for enterprise scalability.

```text
safetix-backend/
├── app/
│   └── main.py          # API Controller,Database Connection & Route Logic
├── tests/
│   └── stress_test.py   # Multi-threaded Load Testing Suite
├── assets/              # Documentation Images
├── requirements.txt     # Dependency Management
└── README.md            # Technical Documentation
