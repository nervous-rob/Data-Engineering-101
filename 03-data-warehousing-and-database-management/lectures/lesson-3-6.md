# Lesson 3.6: Non-Relational Databases

## Navigation
- [← Back to Lesson Plan](../3.6-non-relational-databases.md)
- [← Back to Module Overview](../README.md)

## Overview
Non-relational databases (often called NoSQL databases) have emerged as powerful alternatives to traditional relational systems for specific use cases and data patterns. This lesson explores the various types of non-relational databases, their characteristics, appropriate use cases, and how they complement traditional data warehousing in a modern data architecture.

## Learning Objectives
- Understand the different types of non-relational databases and their strengths
- Learn when to select a non-relational database over a relational one
- Master data modeling techniques for various NoSQL paradigms
- Develop skills for implementing and optimizing non-relational database solutions

## NoSQL Database Fundamentals

The term "NoSQL" originally meant "non-SQL" or "not only SQL," reflecting these databases' departure from the relational model and SQL query language. Today, NoSQL encompasses a diverse array of database technologies designed for specific data models and use cases.

### Evolution and Motivation

Non-relational databases emerged in response to limitations of traditional relational databases for certain use cases.

#### Historical Context

1. **Early Development**
   - Early systems in the 1960s-1970s (navigational databases)
   - Revival in the early 2000s
   - Driven by web-scale companies (Google, Amazon, Facebook)
   - Response to massive data volume and velocity
   - Need for horizontal scalability

2. **Relational Database Limitations**
   - Rigid schema requirements
   - Scaling challenges (especially write scaling)
   - Impedance mismatch with object-oriented programming
   - Performance bottlenecks with massive datasets
   - Complexity in handling semi-structured data

3. **CAP Theorem Influence**
   - Consistency: All nodes see the same data at the same time
   - Availability: Every request receives a response
   - Partition tolerance: System continues operating despite network partitions
   - Traditional RDBMS: Prioritize consistency and availability
   - Many NoSQL systems: Sacrifice consistency for availability and partition tolerance

#### Key Characteristics

1. **Schema Flexibility**
   - Schema-on-read vs. schema-on-write
   - Dynamic schema evolution
   - Support for semi-structured and unstructured data
   - Varied data formats within the same collection
   - Easier adaptation to changing requirements

2. **Scalability**
   - Horizontal scaling through sharding
   - Distributed architecture
   - Linear performance scaling with additional nodes
   - Masterless designs for write scaling
   - Data replication across nodes

3. **Performance Optimization**
   - Data access pattern optimization
   - Reduced join operations
   - In-memory processing options
   - Purpose-built for specific query patterns
   - Optimized for high write throughput

4. **Consistency Models**
   - Strong consistency: All reads receive most recent write
   - Eventual consistency: Given enough time, all replicas converge
   - Causal consistency: Related operations are seen in correct order
   - Tunable consistency in some databases
   - CAP theorem trade-offs

### NoSQL Database Categories

Non-relational databases are typically classified into several main categories based on their data models and access patterns.

#### Document Stores

Document databases store and retrieve documents, typically in JSON, BSON, or XML format, providing flexibility for varied data structures.

1. **Data Model**
   - Self-contained documents (typically JSON)
   - Collections group similar documents
   - Documents can have different fields
   - Nested document structures
   - Arrays and complex values

2. **Key Characteristics**
   - Schema flexibility
   - Rich query language
   - Secondary indexing
   - Document validation options
   - ACID transactions (in modern implementations)

3. **Popular Implementations**
   - MongoDB
   - Couchbase
   - Amazon DocumentDB
   - Azure Cosmos DB (document API)
   - Firebase Firestore

4. **Ideal Use Cases**
   - Content management systems
   - User profiles and preferences
   - Product catalogs
   - Real-time analytics
   - IoT data collection

#### Key-Value Stores

Key-value databases are the simplest NoSQL databases, storing data as a collection of key-value pairs with efficient lookup by key.

1. **Data Model**
   - Simple key to value mapping
   - Values can be strings, numbers, JSON, or blobs
   - Keys are unique within a namespace
   - Minimal metadata
   - Designed for high-speed lookups

2. **Key Characteristics**
   - Extreme scalability
   - Very low latency
   - Simple data model
   - Limited query capabilities
   - Typically distributed by key range

3. **Popular Implementations**
   - Redis
   - Amazon DynamoDB
   - Riak
   - Aerospike
   - Memcached

4. **Ideal Use Cases**
   - Caching
   - Session storage
   - User preferences
   - Real-time leaderboards
   - High-speed counters

#### Column-Family Stores

Column-family databases store data in column families, groups of related data often accessed together, optimized for queries over large datasets.

1. **Data Model**
   - Tables containing row keys
   - Column families group related columns
   - Sparse data storage (only populated columns)
   - Time-versioned values
   - Wide-row design

2. **Key Characteristics**
   - High write throughput
   - Excellent scalability for large datasets
   - Optimized for specific access patterns
   - Column-oriented storage
   - Tunable consistency

3. **Popular Implementations**
   - Apache Cassandra
   - HBase
   - ScyllaDB
   - Google Bigtable
   - Azure Cosmos DB (Cassandra API)

4. **Ideal Use Cases**
   - Time-series data
   - IoT sensor data
   - Recommendation engines
   - Fraud detection systems
   - High-volume write applications

#### Graph Databases

Graph databases focus on relationships between entities, using nodes, edges, and properties to represent and store data.

1. **Data Model**
   - Nodes (entities with properties)
   - Edges (relationships with properties)
   - Labels and types for categorization
   - Properties on both nodes and edges
   - Native graph storage

2. **Key Characteristics**
   - Relationship-focused queries
   - Traversal efficiency
   - Pattern matching capabilities
   - Declarative query languages
   - Visualization capabilities

3. **Popular Implementations**
   - Neo4j
   - ArangoDB
   - Amazon Neptune
   - JanusGraph
   - Azure Cosmos DB (Gremlin API)

4. **Ideal Use Cases**
   - Social networks
   - Recommendation engines
   - Fraud detection
   - Knowledge graphs
   - Network and IT operations

#### Time-Series Databases

Time-series databases are optimized for data points collected, monitored, or tracked over time.

1. **Data Model**
   - Timestamp as primary dimension
   - Metrics, events, or measurements
   - Tags or labels for categorization
   - Retention policies
   - Downsampling capabilities

2. **Key Characteristics**
   - Time-based organization
   - High write throughput
   - Efficient storage compression
   - Time-based query optimization
   - Built-in aggregation functions

3. **Popular Implementations**
   - InfluxDB
   - TimescaleDB
   - Prometheus
   - Kdb+
   - Amazon Timestream

4. **Ideal Use Cases**
   - IoT device monitoring
   - Financial market data
   - Application performance monitoring
   - Industrial telemetry
   - User behavior analytics

#### Multi-Model Databases

Multi-model databases support multiple data models within a single, integrated backend.

1. **Data Model**
   - Multiple models (document, graph, key-value, etc.)
   - Unified query language
   - Consistent transaction semantics
   - Integrated storage
   - Common API

2. **Key Characteristics**
   - Versatility across use cases
   - Reduced data silos
   - Simplified architecture
   - Common security model
   - Unified management

3. **Popular Implementations**
   - ArangoDB
   - Azure Cosmos DB
   - OrientDB
   - FaunaDB
   - Couchbase

4. **Ideal Use Cases**
   - Complex applications with varied data needs
   - Microservices architectures
   - Unified operational and analytical access
   - Reducing database proliferation
   - Future-proofing data architecture

## Detailed Exploration of NoSQL Types

### Document Databases Deep Dive

Document databases provide a flexible, JSON-like data model that aligns well with modern application development.

#### MongoDB Architecture

1. **Core Components**
   - mongod: Primary database process
   - Collections: Groups of related documents
   - Indexes: B-tree structures for query optimization
   - Replica sets: For high availability
   - Sharding: For horizontal scaling

2. **Data Distribution**
   - Sharding strategies (hash, range, tag-based)
   - Chunk management
   - Shard balancing
   - Config servers for metadata
   - MongoDB Router (mongos) for query routing

3. **Consistency Model**
   - Read concerns (local, available, majority)
   - Write concerns (w=1, w=majority, etc.)
   - Causal consistency
   - Transactions (multi-document)
   - Read preferences for replica routing

#### Document Data Modeling

1. **Embedding vs. Referencing**
   - Embedding: Nested documents within a document
   - Referencing: Document references similar to foreign keys
   - One-to-one: Usually embedded
   - One-to-many: Embedded for moderate cardinality
   - Many-to-many: Usually referenced

2. **Schema Design Patterns**
   - Polymorphic pattern
   - Attribute pattern
   - Subset pattern
   - Bucket pattern
   - Computed pattern

3. **Index Strategies**
   - Simple indexes
   - Compound indexes
   - Multikey indexes (for arrays)
   - Text indexes
   - Geospatial indexes

#### Query Capabilities

1. **Query Language**
   - JSON-based query syntax
   - Aggregation framework
   - MapReduce operations
   - Text search
   - Geospatial queries

2. **Advanced Features**
   - Change streams for real-time data
   - Atlas Search for full-text search
   - Time-series collections
   - GraphQL integration
   - MongoDB Compass for visual query building

### Key-Value Stores Deep Dive

Key-value stores provide extreme performance for simple operations, with Redis being a prominent example.

#### Redis Architecture

1. **Core Components**
   - In-memory data store
   - Optional persistence (RDB, AOF)
   - Master-replica replication
   - Redis Cluster for sharding
   - Pub/Sub messaging

2. **Data Structures**
   - Strings
   - Lists
   - Sets
   - Sorted sets
   - Hashes
   - Streams
   - HyperLogLog

3. **Advanced Features**
   - Lua scripting
   - Transactions
   - Pipelining
   - Redis modules
   - Eviction policies

#### Data Modeling with Redis

1. **Key Design Principles**
   - Meaningful key names
   - Key namespacing
   - Flat key hierarchy
   - Consistent delimiters
   - Manageable key length

2. **Common Patterns**
   - Caching with expiration
   - Session storage
   - Rate limiting
   - Leaderboards with sorted sets
   - Publish/subscribe messaging

3. **Performance Optimization**
   - Pipelining commands
   - Server-side Lua scripts
   - Redis modules for custom logic
   - Memory optimization
   - Connection pooling

### Column-Family Stores Deep Dive

Column-family databases like Cassandra excel at handling large-scale distributed data with tunable consistency.

#### Cassandra Architecture

1. **Core Components**
   - Ring architecture
   - Vnodes (virtual nodes)
   - Gossip protocol
   - Snitch for topology
   - Consistent hashing

2. **Data Distribution**
   - Partitioners (Murmur3, Random, etc.)
   - Replication strategies
   - Replication factor
   - Consistency levels
   - Hinted handoff

3. **Storage Engine**
   - SSTables
   - Memtables
   - Commit log
   - Bloom filters
   - Compaction strategies

#### Cassandra Data Modeling

1. **Query-First Design**
   - Model around query patterns
   - Denormalization for performance
   - One table per query pattern
   - No joins
   - Wide rows vs. multiple tables

2. **Key Design**
   - Partition key determines data distribution
   - Clustering columns determine sort order
   - Composite keys for complex patterns
   - Careful cardinality planning
   - Avoiding hotspots

3. **Advanced Techniques**
   - Time series modeling
   - Collection columns (sets, lists, maps)
   - Secondary indexes (with limitations)
   - Materialized views
   - SASI and SAI indexes

#### CQL (Cassandra Query Language)

1. **Query Capabilities**
   - CREATE TABLE with rich types
   - SELECT with WHERE on partition key
   - ALLOW FILTERING (but with caution)
   - Lightweight transactions
   - Batch operations

2. **Best Practices**
   - Avoid full table scans
   - Minimize secondary indexes
   - Use prepared statements
   - Control batch size
   - Understand consistency tradeoffs

### Graph Databases Deep Dive

Graph databases excel at relationship-rich data and traversal queries that would be complex in relational systems.

#### Neo4j Architecture

1. **Core Components**
   - Nodes and relationships
   - Properties on both
   - Labels for categorization
   - Native graph storage
   - Index-free adjacency

2. **Clustering and Scalability**
   - Causal clustering
   - Core servers and read replicas
   - Fabric for federation
   - Sharding strategies
   - High availability architecture

3. **Storage Engine**
   - Native graph format
   - Page cache
   - Transaction logs
   - Schema indexes
   - Full-text indexes

#### Graph Data Modeling

1. **Modeling Principles**
   - Nodes represent entities
   - Relationships connect nodes
   - Both have properties
   - Directional relationships
   - Labels categorize nodes

2. **Common Patterns**
   - Intermediate nodes for many-to-many
   - Typed relationships
   - Dynamic relationships
   - Hierarchical structures
   - Time trees

3. **Performance Considerations**
   - Selective relationship types
   - Strategic indexing
   - Query optimization
   - Data loading strategies
   - Memory management

#### Cypher Query Language

1. **Pattern Matching**
   - ASCII art-like syntax
   - Node patterns: (n:Person)
   - Relationship patterns: -[r:KNOWS]->
   - Variable binding
   - Pattern comprehension

2. **Query Capabilities**
   - Path finding algorithms
   - Aggregation functions
   - Subqueries
   - Complex traversals
   - Temporal operations

## NoSQL and Data Warehousing Integration

Non-relational databases often complement, rather than replace, traditional data warehouses in a modern data architecture.

### Hybrid Architectures

1. **Operational Data Layer**
   - NoSQL for operational data
   - Real-time access patterns
   - Data capture and collection
   - Event processing
   - API serving layer

2. **Data Lake Integration**
   - NoSQL as data sources
   - Streaming integration
   - Change data capture
   - Raw data persistence
   - Schema-on-read approach

3. **Analytical Augmentation**
   - Graph analytics in traditional warehouses
   - Document store for semi-structured data
   - Time-series for metrics and monitoring
   - Key-value for caching warehouse results
   - Multi-model for specialized analytics

### ETL/ELT Considerations

1. **Data Extraction**
   - Connector availability
   - Change streams/CDC
   - Bulk extraction methods
   - API limitations
   - Consistency considerations

2. **Transformation Challenges**
   - Schema mapping
   - Type conversion
   - Denormalization/normalization
   - Nested data handling
   - Relationship modeling

3. **Loading Strategies**
   - Bulk loading
   - Incremental updates
   - Real-time synchronization
   - Bi-directional integration
   - Metadata management

### Real-World Integration Patterns

1. **Polyglot Persistence**
   - Right database for right use case
   - Service-oriented database selection
   - Domain-specific data stores
   - Unified access layer
   - Consistent metadata

2. **Lambda Architecture**
   - Batch layer (warehouse)
   - Speed layer (NoSQL)
   - Serving layer (combined view)
   - Eventual consistency
   - Query routing

3. **Data Mesh Approach**
   - Domain-oriented ownership
   - Data-as-product philosophy
   - Federated governance
   - Self-serve infrastructure
   - Polyglot implementation

## NoSQL Implementation Strategies

Implementing non-relational databases requires different approaches than traditional relational systems.

### Selection Criteria

1. **Use Case Analysis**
   - Data structure requirements
   - Query patterns
   - Write/read ratio
   - Consistency needs
   - Scaling projections

2. **Technical Considerations**
   - Performance requirements
   - Latency expectations
   - Throughput needs
   - Data volume
   - Geographical distribution

3. **Operational Factors**
   - Operational expertise
   - Vendor support
   - Community maturity
   - Monitoring tools
   - Integration capabilities

### Implementation Planning

1. **Proof of Concept**
   - Limited scope validation
   - Performance testing
   - Failure testing
   - Integration validation
   - Schema design validation

2. **Deployment Models**
   - Self-hosted vs. managed service
   - Cloud provider offerings
   - Multi-region considerations
   - Development/testing environments
   - Disaster recovery planning

3. **Migration Strategies**
   - Pilot project approach
   - Phased migration
   - Dual write period
   - Cutover planning
   - Rollback capabilities

### Operational Best Practices

1. **Monitoring**
   - Key metrics by database type
   - Performance dashboards
   - Alerting thresholds
   - Log aggregation
   - Query performance tracking

2. **Backup and Recovery**
   - Backup methodologies
   - Point-in-time recovery
   - Consistency guarantees
   - Testing restore procedures
   - Disaster recovery planning

3. **Scaling Operations**
   - Horizontal scaling procedures
   - Rebalancing strategies
   - Scaling triggers
   - Capacity planning
   - Performance impact management

## Activities

### Activity 1: NoSQL Database Selection

For the following scenarios, determine the most appropriate NoSQL database type:

1. **Social Network Application**
   - User profiles with varied attributes
   - Friend relationships and interactions
   - Content sharing and tagging
   - Activity feeds
   - Group memberships

2. **E-Commerce Platform**
   - Product catalog with diverse attributes
   - Customer purchase history
   - Recommendation engine
   - Shopping cart functionality
   - Inventory management

3. **IoT Monitoring System**
   - Sensor data collection at high frequency
   - Time-based analytics
   - Anomaly detection
   - Device management
   - Historical trend analysis

4. **Financial Fraud Detection**
   - Transaction pattern analysis
   - Relationship mapping
   - Real-time scoring
   - Historical investigation
   - Compliance reporting

For each scenario:
- Recommend a primary database type with justification
- Suggest a specific implementation
- Identify key data modeling considerations
- Outline potential challenges
- Propose a high-level architecture

### Activity 2: Document Database Modeling

Design a document model for a content management system with the following requirements:

1. **Content Types**
   - Articles with sections, tags, and media
   - Videos with transcripts and categories
   - Podcasts with episodes and show notes
   - User-generated comments
   - Shared metadata across types

2. **Access Patterns**
   - Content browsing by type, tag, and category
   - Content search by keyword
   - User engagement tracking
   - Related content recommendations
   - Content performance analytics

Tasks:
1. Create a document model with sample documents
2. Design collections and relationships
3. Define indexing strategy
4. Create example queries for common operations
5. Explain embedding vs. referencing decisions

## Best Practices

### Database Selection and Design

1. **Choose Based on Data Access Patterns**
   - Analyze how data will be accessed
   - Consider write vs. read ratios
   - Evaluate query complexity
   - Assess relationship importance
   - Determine consistency requirements

2. **Data Modeling Guidelines**
   - Model for the query, not the entity
   - Embrace denormalization when appropriate
   - Design for scalability from the start
   - Consider future access patterns
   - Document modeling decisions

3. **Schema Evolution**
   - Plan for schema changes
   - Version documents/entities
   - Implement migration strategies
   - Test with production-like data
   - Document schema history

### Performance Optimization

1. **Indexing Strategy**
   - Index based on query patterns
   - Avoid over-indexing
   - Monitor index usage
   - Compound indexes for complex queries
   - Regular index maintenance

2. **Query Optimization**
   - Monitor slow queries
   - Use explain plans
   - Optimize data access patterns
   - Leverage database-specific features
   - Consider caching strategies

3. **Scaling Considerations**
   - Design for horizontal scaling
   - Understand sharding implications
   - Implement proper partition keys
   - Test with realistic workloads
   - Plan for growth

### Operational Excellence

1. **Monitoring and Alerting**
   - Database-specific monitoring
   - Performance metrics tracking
   - Capacity planning
   - Automated alerting
   - Trend analysis

2. **Backup and Recovery**
   - Regular backup procedures
   - Point-in-time recovery options
   - Cross-region replication
   - Recovery testing
   - Disaster recovery planning

3. **Security Implementation**
   - Authentication mechanisms
   - Fine-grained authorization
   - Encryption (at rest and in transit)
   - Audit logging
   - Compliance controls

## Resources

- Books:
  - "NoSQL Distilled" by Pramod J. Sadalage and Martin Fowler
  - "Seven Databases in Seven Weeks" by Luc Perkins, Eric Redmond, and Jim Wilson
  - "MongoDB: The Definitive Guide" by Shannon Bradshaw, Eoin Brazil, and Kristina Chodorow
  - "Cassandra: The Definitive Guide" by Jeff Carpenter and Eben Hewitt
  - "Graph Databases" by Ian Robinson, Jim Webber, and Emil Eifrem

- Online Resources:
  - [MongoDB University](https://university.mongodb.com/)
  - [Redis Documentation](https://redis.io/documentation)
  - [Apache Cassandra Documentation](https://cassandra.apache.org/doc/latest/)
  - [Neo4j Developer Resources](https://neo4j.com/developer/)
  - [DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)

## Conclusion

Non-relational databases offer powerful alternatives to traditional relational systems for specific use cases and data models. By understanding their strengths, limitations, and appropriate applications, data engineers can select the right tool for each job and create more effective, scalable data architectures.

As we move to the next lesson, we'll explore data warehouse operations, focusing on the ongoing management, maintenance, and optimization tasks required to keep data warehouse systems running efficiently and reliably. 