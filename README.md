# Retail AI Assistant — Architecture Document

## 1. Overview

This project implements a simulation-based Retail AI Assistant capable of performing two roles:

* **Personal Shopper (Revenue Agent)** — recommends products based on user preferences and constraints
* **Customer Support Assistant (Operations Agent)** — evaluates return eligibility using defined business policies

The system follows a **hybrid agentic architecture**, where:

* A language model is used for **intent detection and explanation**
* Deterministic Python functions (tools) handle **data retrieval and business logic**

This design ensures both flexibility (natural language understanding) and reliability (rule-based decisions).

---

## 2. System Architecture

The architecture is divided into three main layers:

---

### 2.1 Agent Layer (LLM + Orchestration)

The agent is implemented using a Groq-hosted LLM (`llama-3.1-70b-versatile`) and performs:

* Intent classification (shopping vs return)
* Response generation (human-like explanations)
* Orchestration of tool calls

Since Groq does not natively support structured tool calling, a **manual orchestration layer** is implemented:

* The LLM determines intent
* Python logic selects and executes the appropriate tool
* The LLM then explains the tool output

This creates a controlled and interpretable agent pipeline.

---

### 2.2 Tool Layer (Deterministic Functions)

The system defines four core tools:

* `search_products(filters)`
* `get_product(product_id)`
* `get_order(order_id)`
* `evaluate_return(order_id)`

Each tool:

* Operates on real CSV datasets
* Applies deterministic filtering or rule-based logic
* Returns structured data

#### Product Recommendation Logic

The `search_products` tool:

* Filters by size availability
* Applies price constraints
* Prioritizes sale items
* Uses `bestseller_score` for ranking

#### Return Evaluation Logic

The `evaluate_return` tool enforces business rules such as:

* Clearance items → not returnable
* Sale items → limited return window (store credit only)
* Vendor exceptions (e.g., exchange-only or extended window)
* Standard return window → 14 days

All decisions are computed programmatically, ensuring consistency.

---

### 2.3 Data Layer

The system uses three data sources:

* **Product Inventory CSV** — product attributes, tags, stock, pricing
* **Orders CSV** — purchase records
* **Policy File** — return and exchange rules

These are loaded into memory using Pandas and accessed exclusively through tool functions.

---

## 3. Design Decisions

---

### 3.1 Hybrid Agent Design

Due to the lack of native tool-calling support in Groq, a hybrid design is used:

* LLM handles **understanding and explanation**
* Python tools handle **execution and validation**

This avoids over-reliance on the model while still leveraging its reasoning ability.

---

### 3.2 Separation of Concerns

The architecture separates:

* **Intent detection (LLM)**
* **Data retrieval (tools)**
* **Business rules (tool logic)**

Benefits:

* Easier debugging
* Improved reliability
* Clear system boundaries

---

### 3.3 Deterministic Business Logic

All critical operations (especially return decisions) are rule-based:

* No LLM involvement in eligibility decisions
* Policies are strictly enforced via code

This ensures:

* Predictability
* Policy compliance
* No ambiguity

---

## 4. Hallucination Prevention

The system minimizes hallucination through multiple safeguards:

---

### 4.1 Tool-Grounded Responses

All factual outputs come from tools:

* Product recommendations are based on real inventory data
* Return decisions are computed from actual order and policy data

The LLM is only used to **explain**, not generate facts.

---

### 4.2 Restricted Data Access

The LLM:

* Does not directly access CSV data
* Cannot fabricate product or order details

All information flows through controlled tool interfaces.

---

### 4.3 Explicit Error Handling

The system handles edge cases explicitly:

* Invalid order ID → returns clear error message
* Missing product → handled gracefully
* No matching products → user is informed

This prevents incorrect assumptions.

---

### 4.4 Deterministic Return Evaluation

Return eligibility is calculated using fixed rules:

* Based on order date, product flags, and policy
* No probabilistic or generative decision-making

This guarantees correctness.

---

## 5. Tool Selection Strategy

---

### 5.1 Intent Detection

The system classifies user input into:

* **Shopping intent**
* **Return/support intent**

This is performed using the LLM via a classification prompt.

---

### 5.2 Execution Flow

#### Shopping Flow:

1. Detect shopping intent
2. Extract basic filters (size, price, tags)
3. Call `search_products(filters)`
4. Pass results to LLM for explanation

---

#### Support Flow:

1. Detect return intent
2. Extract order ID
3. Call `evaluate_return(order_id)`
4. Use LLM to explain decision

---

### 5.3 Controlled Tool Invocation

* Only predefined tools can be called
* No arbitrary execution
* Tool outputs are validated before explanation

---


## 6. Conclusion

This system achieves a balance between:

* **Intelligence (LLM reasoning)**
* **Control (deterministic tools)**
* **Reliability (policy enforcement)**

By separating reasoning from execution, the architecture ensures that:

* Outputs are grounded in real data
* Business rules are consistently applied
* The system remains transparent and explainable

This makes the solution suitable for real-world retail AI applications.

---

