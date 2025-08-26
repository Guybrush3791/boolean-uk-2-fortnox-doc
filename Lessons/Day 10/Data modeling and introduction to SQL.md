---
argument: Data modelling, SQL intro
section: database
lesson count: "2"
ex count: "2"
---
# Data modeling and introduction to SQL
## Lessons
### PostGresSQL Installation
![[Containerized Database Strategies|1000]]
### Data Modelling
[[Data Modelling.pdf|Slides]]

#### Learning Objectives
- Understand **what data modelling is**
- Follow the **data-modelling process** step-by-step
- Distinguish the **three stages** of data modelling
- Learn how to create an **Entity Relationship (ER) Model**

#### Introduction
Data modelling is the practice of planning and illustrating how information will be stored so it meets business needs. A good model shows:
- The kinds of data collected
- Relationships among those data
- How data can be grouped, formatted, and attributed  

Done well, data modelling:
- Reduces development errors  
- Keeps documentation and design consistent  
- Boosts application and database performance  
- Simplifies data mapping across the organisation  
- Improves communication between developers and business-intelligence teams  

#### Data-Modelling Process
1. **Identify entities** – list every thing, event, or concept to be represented.  
2. **Identify key properties** – note the unique attributes that distinguish each entity.  
3. **Identify relationships** – draft how entities relate to one another.  
4. **Map attributes completely** – be sure every required attribute is attached to the correct entity.  
5. **Assign keys & decide normalisation** – choose primary/foreign keys and the degree of normalisation to avoid duplicate data.

#### Stages of Data Modelling
- **Conceptual model** – big-picture view: what the system contains, how it’s organised, and the business rules involved. Notation is kept simple.  

![[Data Modelling - Conceptual|500]]

- **Logical model** – adds detail: attributes, data types, lengths, and all relationships, still independent of any specific technology.  

![[Data Modelling - Logical|700]]

- **Physical model** – a production-ready blueprint: tables plus primary and foreign keys that maintain relationships.

![[Data Modelling - Physical|900]]

#### Creating an Entity Relationship (ER) Model
ER diagrams formally map entities and the relationships between them. Tools such as [draw.io](https://app.diagrams.net/) make it easy to build these visual models before implementation.

#### Practice
Try designing ER models for real-world systems such as:
- Twitter  
- Amazon  
- GitHub  

### Intro SQL
[[Intro SQL.pdf|Slides]]

#### Learning Objectives
- Differentiate **relational** vs **non-relational** databases and know Postgres is relational.  
- Describe SQL as the language used to **query relational databases**.  
- Recognise a database as a specialised **file-system** managed by a server.  
- Create and drop **databases** and **tables**.  
- Define **tables, rows (records), columns (fields)** and **primary keys**.  
- Use SQL to **INSERT** and **SELECT** data.

#### Introduction
A database acts as a *single source of truth* so every user sees consistent, up-to-date data.  
Applications talk to the database via SQL, while an application server enforces business rules through an API.

#### Database Setup
- Postgres can run **locally** or in the **cloud** (e.g. AWS, GKE...)
- We’ll practise on a **local** instance for easy access

#### Data Representation

![[Intro SQL - Datbase ER|1000]]

#### Creating Tables
Define table name, column, data type and constraints 
```sql
-- Users table
CREATE TABLE IF NOT EXISTS users (
id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
name VARCHAR(20) NOT NULL,
email VARCHAR(50) NOT NULL UNIQUE
);

-- Products table
CREATE TABLE IF NOT EXISTS products (
id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
name VARCHAR(50) NOT NULL,
price DECIMAL(10,2) NOT NULL,
discount BOOLEAN NOT NULL DEFAULT FALSE
);

-- Orders table (header for each order, linked to a user)
CREATE TABLE IF NOT EXISTS orders (
id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
userId INT NOT NULL,
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
CONSTRAINT fk_user FOREIGN KEY (userId) REFERENCES users(id)
);

-- Order_items table (links orders to multiple products, implementing the many-to-many)
CREATE TABLE IF NOT EXISTS order_items (
id integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Optional, but useful for unique item identification
orderId INT NOT NULL,
productId INT NOT NULL,
quantity INT NOT NULL CHECK (quantity > 0), -- Ensures positive quantity
CONSTRAINT fk_order FOREIGN KEY (orderId) REFERENCES orders(id),
CONSTRAINT fk_product FOREIGN KEY (productId) REFERENCES products(id),
UNIQUE (orderId, productId) -- Optional: Prevents duplicate products in the same order
);
```

Key points:  
- **SERIAL** auto-increments.  
- **VARCHAR(n)** limits string length.  
- Constraints: **PRIMARY KEY**, **NOT NULL**, **UNIQUE**.

#### Adding Data (CREATE)
```sql
-- Insert sample data into users table
INSERT INTO users (name, email)
VALUES 
  ('Alice Smith', 'alice@example.com'),
  ('Bob Johnson', 'bob@example.com'),
  ('Charlie Davis', 'charlie@example.com');

-- Insert sample data into products table
INSERT INTO products (name, price, discount)
VALUES 
  ('Laptop', 999.99, TRUE),
  ('Smartphone', 599.99, FALSE),
  ('Headphones', 149.99, TRUE),
  ('Keyboard', 49.99, FALSE);

-- Insert sample data into orders table (assuming user IDs 1, 2, 3 were auto-generated above)
INSERT INTO orders (userId)
VALUES 
  (1),  -- Order for Alice
  (2);  -- Order for Bob

  -- Insert sample data into order_items table (assuming order IDs 1, 2 and product IDs 1-4 were auto-generated above)
INSERT INTO order_items (orderId, productId, quantity)
VALUES 
  (1, 1, 1),  -- Laptop in Alice's order
  (1, 3, 2),  -- Two Headphones in Alice's order
  (2, 2, 1),  -- Smartphone in Bob's order
  (2, 4, 1);  -- Keyboard in Bob's order
```

#### Querying Data (READ)
```sql
-- all columns, all rows  
SELECT * 
FROM products;

-- only name and price  
SELECT name, price 
FROM products;

-- add constraints  
SELECT name, price  
FROM products  
WHERE price < 150  
ORDER BY price ASC  
LIMIT 1;
```

#### Modifying Tables & Data
- **DROP TABLE users;** – remove table and all data.  
- **TRUNCATE users;** – delete data, keep structure.  
- **ALTER TABLE** – add/remove/modify columns (covered later).

#### Viewing Table Metadata
```sql
SELECT table_name, column_name, data_type  
FROM information_schema.columns  
WHERE table_name = 'products';
```

#### Relationships & JOINS
Tables connect through **foreign keys**; SQL combines them with JOINS:

```sql
SELECT 
    u.id AS user_id, 
    u.name AS user_name, 
    SUM(p.price * oi.quantity) AS total_order_price
FROM 
    users u
JOIN 
    orders o ON u.id = o.userId
JOIN 
    order_items oi ON o.id = oi.orderId
JOIN 
    products p ON oi.productId = p.id
GROUP BY 
    u.id
ORDER BY 
    total_order_price DESC;
```

#### Practice
1. Create a new database in *DBEaver*
2. Run CREATE TABLE for `users`.  
3. INSERT a few rows, then SELECT them.  
4. TRUNCATE the table, verify it’s empty.  
5. DROP the table and recreate it to reinforce the workflow.  

For extra drills, complete SQLBolt lessons from “SELECT queries 101” through “SQL Review: Simple SELECT Queries”.

## Excercise
### Data Modelling
[Repository link](https://github.com/boolean-uk/java-data-modelling.git)
![[Repository/Day 10/Ex/1 - Data Modelling/README]]
### Intro SQL
[Repository link](https://github.com/boolean-uk/api-sql-intro.git)
![[Repository/Day 10/Ex/2 - Api SQL Intro/README]]