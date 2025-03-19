# Lesson 3.2: Data Warehouse Design

## Navigation
- [← Back to Lesson Plan](../3.2-data-warehouse-design.md)
- [← Back to Module Overview](../README.md)

## Overview
Data warehouse design is a critical foundation for successful analytics initiatives. This lesson explores schema design patterns, dimensional modeling techniques, and optimization strategies that enable efficient data retrieval and analysis while maintaining data integrity and adaptability.

## Learning Objectives
- Master various data warehouse schema designs
- Understand normalization and denormalization trade-offs
- Learn dimensional modeling techniques and best practices
- Develop skills for optimizing schema performance

## Schema Design Patterns

Data warehouse schema design focuses on organizing data for analytical queries rather than transaction processing. Several well-established patterns have emerged over decades of data warehousing practice.

### Star Schema

The star schema is the most common dimensional modeling pattern, characterized by a central fact table connected to multiple dimension tables, forming a star-like structure.

#### Components

1. **Fact Table**
   - Contains business measurements or metrics
   - Includes foreign keys to dimension tables
   - Typically contains numeric, additive values
   - Usually has a composite primary key made of dimension foreign keys
   - Often has many rows (millions to billions)

2. **Dimension Tables**
   - Contain descriptive attributes about business entities
   - Each has a single primary key that links to the fact table
   - Contains textual, descriptive data
   - Relatively few rows compared to fact tables
   - Often denormalized for query simplicity

#### Example Star Schema

![Star Schema](https://i.imgur.com/VyQ8b3T.png)

In this example:
- The central `Sales_Fact` table contains measures like quantity sold and revenue
- It connects to dimensions like `Date_Dim`, `Product_Dim`, `Store_Dim`, and `Customer_Dim`
- Each dimension provides context for analyzing the facts

#### Advantages
- Simple, intuitive structure
- Optimized for OLAP queries
- Fewer joins required for complex queries
- Better query performance
- Easier for business users to understand
- Simplified BI tool integration

#### Disadvantages
- Some data redundancy in denormalized dimensions
- Can require more storage space
- May have data integrity challenges
- Updates to dimension data can be complex

### Snowflake Schema

The snowflake schema is a variation of the star schema where dimension tables are normalized into multiple related tables, forming a snowflake pattern.

#### Components

1. **Fact Table**
   - Same structure and purpose as in star schema

2. **Dimension Tables**
   - Normalized into multiple tables
   - Hierarchical relationships between dimension tables
   - Reduced redundancy compared to star schema

#### Example Snowflake Schema

![Snowflake Schema](https://i.imgur.com/G3k0Zvl.png)

In this example:
- The `Product_Dim` from a star schema is normalized into `Product`, `Category`, and `Manufacturer` tables
- The `Store_Dim` is normalized into `Store`, `City`, and `Region` tables

#### Advantages
- Reduced data redundancy
- Better enforcement of data integrity
- Smaller dimension tables
- More accurate representation of hierarchical relationships
- Can save storage space

#### Disadvantages
- More complex queries requiring additional joins
- Typically slower query performance
- More complex for business users to understand
- More difficult to maintain

### Galaxy Schema (Fact Constellation)

The galaxy schema (also called fact constellation) contains multiple fact tables that share dimension tables, allowing for integrated analysis across business processes.

#### Components

1. **Multiple Fact Tables**
   - Represent different business processes
   - May have different granularity
   - Share common dimension tables

2. **Conformed Dimensions**
   - Shared across multiple fact tables
   - Consistent keys, attributes, and values
   - Enable cross-process analysis

#### Example Galaxy Schema

![Galaxy Schema](https://i.imgur.com/IXfQZSN.png)

In this example:
- Multiple fact tables (`Sales_Fact`, `Inventory_Fact`) share dimensions
- `Date_Dim`, `Product_Dim`, and `Store_Dim` are conformed dimensions
- Each fact table may have additional dimensions not shared with others

#### Advantages
- Supports enterprise-wide analytics
- Enables cross-functional analysis
- Promotes reuse of dimension definitions
- Provides consistent reporting across the organization

#### Disadvantages
- More complex to design and implement
- Requires enterprise agreement on dimension definitions
- More challenging to maintain
- May involve performance trade-offs

## Normalization vs. Denormalization

Data warehouse design involves strategic decisions about normalizing or denormalizing data structures, with different considerations than operational database design.

### Normalization Principles

Normalization is the process of organizing data to reduce redundancy and improve data integrity.

#### Levels of Normalization

1. **First Normal Form (1NF)**
   - Eliminate repeating groups
   - Create separate tables for related data
   - Identify each set of related data with a primary key

2. **Second Normal Form (2NF)**
   - Meet all requirements of 1NF
   - Remove subsets of data that apply to multiple rows
   - Create separate tables for these subsets
   - Relate these tables with foreign keys

3. **Third Normal Form (3NF)**
   - Meet all requirements of 2NF
   - Remove fields that don't depend on the key
   - Create separate tables for these fields

#### Benefits of Normalization
- Minimizes redundancy
- Reduces update anomalies
- Ensures data consistency
- Saves storage space
- Facilitates database maintenance

### Denormalization Strategies

Denormalization is the strategic introduction of redundancy to improve query performance.

#### Common Denormalization Techniques

1. **Pre-joined Tables**
   - Combining tables that are frequently joined
   - Reduces join operations at query time

2. **Materialized Aggregates**
   - Precalculated summary tables
   - Avoids expensive calculations at query time

3. **Redundant Attributes**
   - Duplicating attributes across related tables
   - Reduces need for joins

4. **Derived Attributes**
   - Storing calculated values
   - Avoids recalculation at query time

#### Benefits of Denormalization
- Improved query performance
- Reduced join complexity
- Simplified query writing
- Better report generation speed
- Enhanced user experience

### Finding the Right Balance

Most data warehouses use a strategic combination of normalization and denormalization:

- **Core Approach**: Start with a clear dimensional model (star or snowflake)
- **Storage Tier**: Consider more normalized structures for the persistence layer
- **Query Tier**: Create denormalized views or aggregates for performance
- **Business Need**: Let query patterns and reporting needs guide the approach
- **Technical Constraints**: Consider system capabilities and data volumes

## Dimensional Modeling Techniques

Dimensional modeling is a design technique optimized for data warehousing and business intelligence.

### Dimension Table Design

Dimensions provide the context for analyzing numeric facts.

#### Types of Dimensions

1. **Conformed Dimensions**
   - Consistent dimensions shared across multiple fact tables
   - Example: A customer dimension used by both sales and support fact tables
   - Enables cross-process analysis
   - Requires enterprise agreement on definitions

2. **Role-Playing Dimensions**
   - Same dimension used multiple times in the same fact table with different roles
   - Example: Date dimension used as order_date, ship_date, and delivery_date
   - Implementation options:
     - Multiple foreign keys to the same dimension table
     - Multiple views on the same dimension table

3. **Junk Dimensions**
   - Composite dimensions combining multiple low-cardinality flags or attributes
   - Example: Combining order_status, payment_method, and delivery_method
   - Reduces the number of dimensions in the schema
   - Simplifies the fact table structure

4. **Degenerate Dimensions**
   - Dimensional values stored in the fact table without a separate dimension table
   - Example: Order number or invoice number
   - Typically used for transaction identifiers
   - Not used for filtering or grouping

#### Slowly Changing Dimensions (SCDs)

Managing changes to dimension attributes over time is a critical aspect of dimensional modeling.

1. **Type 0 SCD**
   - Fixed dimensions that never change
   - Example: Birth date, original credit score
   - Implementation: No special handling required

2. **Type 1 SCD**
   - Overwrite old values with new values
   - No history is maintained
   - Example: Correcting errors in address
   - Implementation: Simple updates to dimension records

3. **Type 2 SCD**
   - Maintain full history by adding new rows
   - Example: Customer address changes
   - Implementation:
     - Add effective_date, expiration_date fields
     - Add current_flag field
     - Add surrogate keys
     - Create new row for each change

4. **Type 3 SCD**
   - Keep limited history with previous value columns
   - Example: Previous and current sales region
   - Implementation:
     - Add previous_value columns
     - Add effective_date field
     - Update both current and previous values

5. **Type 4 SCD**
   - Use history tables to track all changes
   - Example: Complete customer profile history
   - Implementation:
     - Maintain current values in main dimension
     - Store all historical values in separate history table

6. **Type 6 SCD**
   - Hybrid approach combining Types 1, 2, and 3
   - Example: Critical dimensions requiring full history and fast access
   - Implementation:
     - Type 2 approach with new rows
     - Type 1 current value columns
     - Type 3 previous value columns

### Fact Table Design

Fact tables contain the measurements or metrics of business processes.

#### Types of Fact Tables

1. **Transaction Fact Tables**
   - One row per transaction event
   - Highest level of detail (atomic)
   - Example: Individual sales transactions
   - Typically largest tables in the warehouse
   - Provides maximum flexibility for analysis

2. **Periodic Snapshot Fact Tables**
   - Regular snapshots at consistent time intervals
   - Example: Monthly account balances
   - Captures state at specific points in time
   - Good for trend analysis over consistent periods

3. **Accumulating Snapshot Fact Tables**
   - One row per process instance, updated as events occur
   - Example: Order fulfillment with multiple milestones
   - Multiple date dimensions for each milestone
   - Records complete process lifecycle
   - Good for process performance analysis

4. **Factless Fact Tables**
   - Fact tables without numeric measures
   - Record relationships or events
   - Examples:
     - Student attendance (student, class, date)
     - Product promotions (product, promotion, date)
   - Used for coverage or event analysis

#### Types of Measures

1. **Additive Measures**
   - Can be summed across all dimensions
   - Examples: Sales amount, quantity sold
   - Most common and useful measures

2. **Semi-Additive Measures**
   - Can be summed across some dimensions but not all
   - Examples: Account balances (additive across accounts, not time)
   - Often found in snapshot fact tables

3. **Non-Additive Measures**
   - Cannot be summed meaningfully
   - Examples: Ratios, percentages, unit prices
   - Require special handling in analytics

## Schema Optimization Strategies

Optimizing schema design for query performance involves several techniques.

### Indexing Strategies

1. **Bitmap Indexes**
   - Efficient for low-cardinality columns
   - Common in data warehousing
   - Good for dimension key columns in fact tables

2. **B-tree Indexes**
   - Effective for high-cardinality columns
   - Used for primary keys and unique columns
   - Good for selective queries

3. **Covering Indexes**
   - Include all columns needed by a query
   - Eliminate need to access base tables
   - Optimize specific query patterns

### Partitioning

1. **Horizontal Partitioning**
   - Splitting tables based on data values
   - Common partitioning keys:
     - Date/time (most common)
     - Region/geography
     - Business unit
   - Benefits:
     - Improved query performance
     - Easier maintenance
     - Parallel processing

2. **Vertical Partitioning**
   - Splitting tables by columns
   - Less common in warehousing
   - Used for very wide tables with seldom-used columns

### Aggregation

1. **Materialized Aggregates**
   - Precalculated summary tables
   - Examples:
     - Daily, monthly, quarterly summaries
     - Regional, categorical rollups
   - Benefits:
     - Dramatically improved query performance
     - Reduced resource usage
     - Better user experience

2. **Aggregate Navigation**
   - Automatically selecting appropriate aggregates
   - Query rewriting to use aggregates
   - Can be transparent to end users

## Activities

### Activity 1: Schema Design Comparison

Analyze the following business scenario and compare star, snowflake, and galaxy schema approaches:

A retail company needs to analyze:
- Sales performance (revenue, units, profit) by product, store, customer, and time
- Inventory levels and movements by product, store, and time
- Marketing campaign effectiveness by product, promotion, customer segment, and time

For each schema type:
1. Sketch the basic structure showing tables and relationships
2. Identify 3-5 key advantages for this specific scenario
3. Identify 3-5 key disadvantages or challenges
4. Make a recommendation with justification

### Activity 2: Dimensional Modeling Workshop

Design a dimensional model for a healthcare analytics system with these requirements:

1. Track patient visits including:
   - Visit date, time, and duration
   - Reason for visit
   - Diagnoses and procedures
   - Treating physicians and departments
   - Charges and payments

2. Support these analyses:
   - Patient visit patterns over time
   - Physician productivity and specialization
   - Department utilization
   - Procedure costs and frequency
   - Diagnosis trends

Design tasks:
1. Identify fact and dimension tables
2. Determine appropriate fact table grain
3. Design dimension tables with attributes
4. Identify slowly changing dimensions and recommend SCD types
5. Document expected query patterns

## Best Practices

### Schema Design Guidelines

1. **Start with Business Requirements**
   - Design around key business questions
   - Understand reporting and analytics needs
   - Consider future analytical requirements

2. **Choose the Right Level of Detail**
   - Use atomic data when possible
   - Balance detail with performance needs
   - Consider query patterns and frequency

3. **Standardize Naming Conventions**
   - Consistent, clear, and descriptive names
   - Business-friendly terminology
   - Indicate fact vs. dimension tables

4. **Design for Flexibility**
   - Anticipate changing business needs
   - Plan for new data sources
   - Consider long-term maintenance

5. **Document Thoroughly**
   - Define business meanings of all elements
   - Document rationale for design decisions
   - Maintain data dictionary and metadata

### Performance Optimization

1. **Balance Normalization and Denormalization**
   - Normalize for data integrity
   - Denormalize for query performance
   - Consider hybrid approaches

2. **Implement Effective Indexing**
   - Index dimension keys in fact tables
   - Create appropriate indexes for common queries
   - Avoid over-indexing

3. **Use Partitioning Strategically**
   - Partition large fact tables by date
   - Align partitioning with query patterns
   - Consider partition elimination

4. **Employ Aggregation**
   - Create materialized views for common aggregates
   - Refresh aggregates efficiently
   - Document aggregation logic

5. **Optimize Join Performance**
   - Minimize complex joins
   - Use appropriate join techniques
   - Consider column order in joins

## Resources

- Books:
  - "The Data Warehouse Toolkit" by Ralph Kimball and Margy Ross
  - "Star Schema: The Complete Reference" by Christopher Adamson
  - "Agile Data Warehouse Design" by Lawrence Corr and Jim Stagnitto

- Online Resources:
  - [Kimball Group Design Tips](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/)
  - [Data Warehouse Design Best Practices](https://www.snowflake.com/blog/best-practices-for-data-warehouse-design/)
  - [Dimensional Modeling Fundamentals](https://docs.microsoft.com/en-us/power-bi/guidance/star-schema)

## Conclusion

Effective data warehouse design is foundational to successful business intelligence and analytics initiatives. By applying dimensional modeling techniques, choosing appropriate schema patterns, and implementing performance optimization strategies, you can create data warehouses that deliver both analytical flexibility and query performance.

As we move to the next lesson, we'll shift from design to implementation, exploring the practical aspects of bringing your data warehouse designs to life in various platforms and environments. 