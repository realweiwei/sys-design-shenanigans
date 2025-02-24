# CAP Theorem
- In a **distributed system** (nodes that share data and communicate over a network), you can only guarantee **two out of three** properties across a read/write operation.
## Consistency (C)
- Every read receives the most recent write or an error. 
- In formal terms (atomic/ linearizable consistency), the system behaves as if all operations occur in a single, sequential order. 
## Availability (A)
- Every request receives a (non-error) response, without guaranteeing it contains the latest write. 
- The system always responds eventually (no timeouts or errors), but the data might be stale. 
## Partition Tolerance (P)
- The system continues to function despite network partitions (message loss or unreachability between node).
# Consistency Patterns
## Weak Consistency
- After a write, reads may or may not see it immediately.
- A "best effort" approach is used. 
	- The system tries to propagate updates and keep data in sync as quickly as possible, but it does **not** guarantee that every read will see the lates write.
- Commonly used in real-time use cases: 
	- VoIP
		- Voice over Internet Protocol
		- VoIP is a technology that allows users to make voice calls using a broadband internet connection rather than a traditional phone line.
	- video chat
	- real-time multiplayer game
- Example
	- Missing a few seconds of a phone call when reception drops; once connection is regained, the missed audio isn't replayed.
## Eventual Consistency
- After a write, reads will eventually see the updated data (typically within milliseconds).
- Data replication is asynchronous.
	- **Data replication:*** The process of copying and maintaining data across multiple nodes, servers, or locations.
		- Synchronous Replication: Updates happen in real-time before conforming a write. 
		- Async Replication: Writes are confirmed immediately, and updates propagate later.
- Commonly used in **highly available** systems such as DNS and email.
	- Highly Available Applications:
		- They aim to minimize downtime and ensure continuous service operation, often measured in terms of "number of 9s" of availability (e.g., 99.9% or 99.99%).
		- They are designed with redundancy and fail-over mechanisms, so that if one server or component fails, another can take over with minimal (or zero) disruption.
	- DNS (Domain Name System)
		- DNS is often described as the "phonebook of the internet."
		- It translates human-readable domain names (e.g., `www.example.com`) into IP addresses (e.g., `192.0.2.1`) that computers use to identify each other on the network.
		- When you type a URL into your browser, your computer asks a DNS server to look up the IP address for that domain. Once it receives the address, it can connect to the correct server and load the website.
## Strong Consistency
- After a write, all reads will see the updated data. 
- Data replication is synchronous. 
- Commonly used in file systems and relational databases (RDBMS), where transactional guarantees are important. 
	- "ACID" in database systems: Atomicity, Consistency, Isolation, Durability
	- Data Integrity:
		- Ensures that data is accurate, consistent, and does not get corrupted or partially updated. 
		- In db, this prevents anomalies such as missing records or inconsistencies when multiple operations happen concurrently.
	- Reliability and Fault Tolerance: 
		- Guarantees that either parts of a transaction are committed (saved) or none are, preventing partial writes if a failure occurs midway.
		- This is crucial in scenarios like power loss, hard failures, or software crashes.
	- Consistency Across Operations: 
		- Ensures that each subsequent operation works on valid, consistent data, even under high concurrency.
		- In file systems, transactional guarantees can maintain a consistent view of files and directories, preventing corruption when multiple processes read and write simultaneously.
	- Predictable Behavior: 
		- Developers can design systems with confidence, knowing that transactions will succeed or fail as an atomic unit. 
# Availability Patterns
## Fail-over
- Used to maintain service availability by having at least one backup system ready to take over.
- Ensures **high availability** and **minimal downtime** if a server or service fails.
### Active-Passive (Master-Slave)
- Heartbeats are sent between the active and passive systems.
	- Usually, the **slave (passive node)** monitors the **master (active node)** by sending or expecting heartbeat signals. If master fails to respond within a configured timeout, the slave promotes itself to become the new master.
	- In some implementations, the master also monitors the slave to verify it is alive and can safely take over if needed. 
	- Heartbeat: 
		- A "heartbeat" is a lightweight signal or message exchanged at regular intervals (e.g., every few seconds).
		- If the master is active, it either sends a heartbeat message to the slave or responds to the slave's ping, indicating it is still operational. 
		- If the salve stops receiving heartbeats (or acknowledgements) from the master for a certain threshold of time or number of attempts, it concludes the master is down. 
		- Once that threshold is crossed, failover logic is triggered, and the slave takes over the master's responsibilities (e.g., by assuming the master's IP address or other critical identifiers.)
- If the active system fails (heartbeat lost), the passive takes over the active's IP and resumes service. 
- Downtime depends on whether passive is "hot" (already running) or "cold" (needs startup) standby. 
- Only the active server handles traffic under normal conditions.
### Active-Active (Master-Master)
- Both systems actively handle traffic, sharing the load.
- Public-facing servers required DNS to know both public UPs; internal-facing servers require application logic to handle multiple servers.
	- Common strategies to decide which IP a connection should be directed to: 
		- DNS-Based Load Balancing (Round Robin or Weighted)
		- Load Balancer in Front
		- Application-Aware Routing
### Disadvantages: 
- Additional hardware and complexity.
- Potential loss of data if the active system fails before data is replicated to the passive system.
## Replication
- Can be configured as **master-slave** or **master-master**.
- Ensures **data redundancy** and often **improved read scalability**.
- Data written on one node (the master) is copied to one or more slaves (replicas).
- Focuses on **keeping data in sync** across multiple nodes or apps.
### Master-Slave
- Writes happen on the master and changes are copied (sync or async) to the salve(s).
- ![[sys-design-shenanigans/attachments/CPSC436r 2.jpeg]]
### Master-Master 
- Both nodes accept writes, and changes are exchanged in both directions to keep data consistent.
- ![[sys-design-shenanigans/attachments/IMG_68C08BEA6FB7-1.jpeg]]
## Availability in Parallel vs. in Sequence
### In Sequence
- The entire service depends on each component to be available.
- If any one component is down, the service fails.
- `Availabilitytotal​=Availability(Foo)×Availability(Bar)`
### In Parallel
- The service can still run as long as at least one component is available.
- `Availabilitytotal​=1−(1−Availability(Foo))×(1−Availability(Bar))`