# Data-Intensive Systems

These notes are a working knowledge bank for data-intensive system design. The goal is to build language for reasoning about trade-offs, choosing the right tools, and understanding how different parts of a system fit together.

## Introduction

- The landscape of technologies for storing and processing data changes quickly, but many of the underlying principles are stable.

- If we understand those principles, we can reason about where each tool fits, how to use it, and which pitfalls to avoid.

- This is not meant to be a tutorial for one specific tool, and it is not meant to be abstract theory disconnected from practice.

- The goal is to study the recurring patterns behind real data systems: how they store data, process changes, serve queries, scale, recover from failures, and make trade-offs.

- The most useful way to study these systems is not only to ask how they work, but why they work that way.

- A recurring theme is that there is no perfect approach. Each design gives us something and costs us something.

## Data-Intensive Applications

- We call an application data-intensive if data management is one of the primary challenges in developing the application.

- We call an application compute-intensive if the primary challenge is to parallelize a very large computation.

- In data-intensive applications, we usually worry more about:
  - storing and processing large volumes of data
  - managing changes to data
  - ensuring consistency in the face of failures and concurrency
  - making sure services are highly available

## Standard Building Blocks of Data-Intensive Applications

- Data-intensive applications are typically built from standard building blocks that provide commonly needed functionality.

- For example, many applications need to:
  - store data so that they, or another application, can find it again later (**databases**)
  - remember the result of an expensive operation to speed up reads (**caches**)
  - allow users to search data by keyword or filter it in various ways (**search indexes**)
  - handle events and data changes as they occur (**stream processing**)
  - periodically process a large amount of accumulated data (**batch processing**)

## Choosing and Combining Approaches

- How do you choose which approach to use?

- There are many kinds of caches, databases, indexes, streams, and batch systems.

- It can also be difficult to combine them when you need to do something a single tool cannot do alone.

- The goal of these notes is to help us ask the right questions, evaluate the trade-offs, and decide which approach best serves a particular application or use case.

- One key challenge is that different people often need to do different things with the same data.

- One team may care about low-latency user requests, while another team may care about historical analysis, reporting, fraud detection, or business decision support.

- If those goals are not explicitly articulated, teams can disagree about the right approach even when they are looking at the same underlying dataset.

- To understand the choices, we can compare several contrasting concepts:
  - operational systems and analytical systems
  - cloud services and self-hosted systems
  - single-node systems and distributed systems
  - business needs and users' rights or expectations

- Not every contrast matters equally for every system. Some questions are lightweight, while others drive the architecture.

## Transaction Processing vs Analytical Processing

- Operational systems are optimized for serving the live application.

- Analytical systems are optimized for exploring, aggregating, and learning from data over time.

- This distinction matters because the same dataset can support very different access patterns.

| Property | Operational systems (OLTP) | Analytical systems (OLAP) |
| --- | --- | --- |
| Main read pattern | Small lookups, often by key or by a narrow filter | Scans, joins, and aggregations across many records |
| Main write pattern | Frequent inserts, updates, and deletes of individual records | Bulk loads, ETL jobs, or event-stream ingestion |
| Human user example | End user using a web or mobile application | Internal analyst exploring data for decisions |
| Machine use example | Checking whether a user action is allowed | Detecting fraud, abuse, or unusual patterns |
| Type of queries | Fixed queries defined by application code | Flexible, ad hoc questions from analysts |
| Query volume | Many small queries | Fewer queries, but each one may be expensive |
| Data represents | Current state of the application | Historical record of events over time |
| Dataset size | Often gigabytes to terabytes | Often terabytes to petabytes |

- The point of this table is not that one system type is better than the other.

- The point is that the workload is different, so the storage layout, query engine, update strategy, and operational trade-offs are often different too.

## Data Warehousing

- Historically, the same database was often used for both transaction processing and analytical queries.

- SQL made this possible because it was flexible enough to support both application queries and analytical queries.

- Over time, many companies moved away from running analytics directly on the OLTP system.

- Instead, they started copying data into a separate database designed for analytical workloads.

- That separate analytical database is called a data warehouse.

- The basic idea is to keep the transactional database focused on serving the live application, while the data warehouse supports reporting, analysis, and decision-making.

- A data warehouse gives analysts a place to query company data without putting load on the online transaction systems.

- Data warehouses store data differently from transaction databases because analytical queries usually need to scan, aggregate, and compare large amounts of data.

- A data warehouse usually contains a read-only copy of data from many online transaction systems across the company.

- Data is moved into the warehouse through a pipeline:
  - extract data from source transaction systems
  - transform it into a schema that is easier to analyze
  - clean up inconsistencies or messy source data
  - load it into the data warehouse

- This process is commonly called extract, transform, load, or ETL.

- Sometimes the order changes, and the data is loaded before it is transformed, but the important idea is that data is copied out of operational systems and prepared for analysis elsewhere.

## Terminology: Frontends and Backends

- Much of data-intensive system design relates to backend development.

- For web applications, the client-side code that runs in a browser is called the frontend.

- The server-side code that handles user requests is called the backend.

- Mobile apps are similar to frontends because they provide user interfaces that usually communicate with a backend over the internet.

- Frontends can store some data locally on the user's device, but the hardest data infrastructure problems usually appear in the backend.

- A frontend usually handles one user's local experience, while a backend manages data on behalf of many users.

- A backend service is often reachable over HTTP, or sometimes WebSocket.

- It usually consists of application code that reads and writes data in one or more databases.

- It may also interact with additional data systems such as caches, message queues, search indexes, stream processors, or batch jobs.

- We can collectively call those backend storage and processing systems data infrastructure.

- Backend application code is often stateless, meaning that when it finishes handling one request, it does not keep that request's state in memory as the durable source of truth.

- Any information that must persist from one request to another needs to be stored either on the client or in server-side data infrastructure.

## Explanation Pattern: Connected Terminology

- Some concepts are hard to explain because they are not isolated definitions.

- The hard part is often explaining a group of connected terms and the boundaries between them.

- A strong explanation pattern is:
  - name the terminology cluster
  - explain why the terms matter in this context
  - define each term in relation to the others
  - clarify the responsibility boundary
  - add concrete examples only when they make the relationship clearer
  - end with the architectural consequence

- This pattern is useful when the goal is not just to memorize vocabulary, but to reason about design decisions.
