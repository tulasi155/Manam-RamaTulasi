-- TASK 1: Conceptual Design using ER Model Concepts

-- Entities: Users, Temples, Tickets, Payments, Admin

-- 1.a Entities and Attributes identified
-- USERS(user_id, user_name, email, phone)
-- TEMPLES(temple_id, temple_name, location)
-- TICKETS(ticket_id, user_id, temple_id, visit_date, booking_date)
-- PAYMENTS(payment_id, ticket_id, amount, payment_mode, status)
-- ADMIN(admin_id, admin_name, email, password)

-- 1.b Relationships:
-- User books Ticket
-- Ticket belongs to Temple
-- Ticket has Payment
-- Admin manages Temple

-- 1.c Cardinality:
-- One User -> Many Tickets
-- One Temple -> Many Tickets
-- One Ticket -> One Payment

-- 1.d Create relations with keys and constraints
CREATE DATABASE temple_ticket_system;
USE temple_ticket_system;

CREATE TABLE users (
  user_id INT AUTO_INCREMENT PRIMARY KEY,
  user_name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE,
  phone VARCHAR(15)
);

CREATE TABLE temples (
  temple_id INT AUTO_INCREMENT PRIMARY KEY,
  temple_name VARCHAR(100),
  location VARCHAR(100)
);

CREATE TABLE tickets (
  ticket_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  temple_id INT,
  visit_date DATE,
  booking_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (temple_id) REFERENCES temples(temple_id)
);

CREATE TABLE payments (
  payment_id INT AUTO_INCREMENT PRIMARY KEY,
  ticket_id INT,
  amount DECIMAL(8,2),
  payment_mode VARCHAR(50),
  status VARCHAR(20),
  FOREIGN KEY (ticket_id) REFERENCES tickets(ticket_id)
);

CREATE TABLE admin (
  admin_id INT AUTO_INCREMENT PRIMARY KEY,
  admin_name VARCHAR(50),
  email VARCHAR(100),
  password VARCHAR(50)
);

-- TASK 2: Hierarchical/Network Model and DDL, DCL operations

-- Domain constraints and check constraints
ALTER TABLE payments
ADD CONSTRAINT chk_payment_status CHECK (status IN ('Success', 'Pending', 'Failed'));

-- Rename relation
RENAME TABLE temples TO temple_details;

-- Create a generalized table (Inheritance Example)
CREATE TABLE persons (
  person_id INT PRIMARY KEY,
  person_name VARCHAR(100)
);

CREATE TABLE devotees (
  devotee_id INT PRIMARY KEY,
  person_id INT,
  FOREIGN KEY (person_id) REFERENCES persons(person_id)
);

-- SQL using DDL & DCL
CREATE USER 'temple_admin'@'localhost' IDENTIFIED BY 'admin123';
GRANT SELECT, INSERT, UPDATE ON temple_ticket_system.* TO 'temple_admin'@'localhost';

-- TASK 3: Using Clauses, Operators, and Functions

-- Insert sample data
INSERT INTO users (user_name, email, phone)
VALUES ('Rama Tulasi', 'rama@gmail.com', '9876543210'),
       ('Pranathi', 'pranathi@gmail.com', '9876501234');

INSERT INTO temple_details (temple_name, location)
VALUES ('Tirupati Temple', 'Tirupati'),
       ('Sabarimala Temple', 'Kerala');

INSERT INTO tickets (user_id, temple_id, visit_date)
VALUES (1, 1, '2025-11-10'), (2, 2, '2025-11-15');

INSERT INTO payments (ticket_id, amount, payment_mode, status)
VALUES (1, 500.00, 'UPI', 'Success'), (2, 750.00, 'Card', 'Pending');

-- Queries with functions
SELECT UPPER(temple_name) AS TempleName, LENGTH(location) AS NameLength FROM temple_details;
SELECT user_name, NOW() AS CurrentTime FROM users;
SELECT COUNT(*) AS TotalTickets, SUM(amount) AS TotalRevenue FROM payments WHERE status='Success';

-- TASK 4: Using Subqueries and Functions

-- Find users who booked temples located in 'Kerala'
SELECT user_name
FROM users
WHERE user_id IN (
  SELECT user_id FROM tickets
  WHERE temple_id IN (SELECT temple_id FROM temple_details WHERE location='Kerala')
);

-- Find maximum payment amount
SELECT MAX(amount) FROM payments;

-- Correlated subquery example
SELECT t.ticket_id, (SELECT temple_name FROM temple_details WHERE temple_id=t.temple_id) AS TempleName
FROM tickets t;

-- TASK 5: Writing Join Queries

-- INNER JOIN
SELECT u.user_name, t.ticket_id, td.temple_name
FROM users u
JOIN tickets t ON u.user_id = t.user_id
JOIN temple_details td ON t.temple_id = td.temple_id;

-- LEFT JOIN
SELECT u.user_name, p.amount
FROM users u
LEFT JOIN tickets t ON u.user_id = t.user_id
LEFT JOIN payments p ON t.ticket_id = p.ticket_id;

-- Recursive query (CTE)
WITH RECURSIVE Numbers AS (
  SELECT 1 AS n
  UNION ALL
  SELECT n+1 FROM Numbers WHERE n < 5
)
SELECT * FROM Numbers;

-- TASK 6: Procedures, Functions, Loops

DELIMITER $$

CREATE PROCEDURE GetTempleRevenue(IN temple INT)
BEGIN
  SELECT td.temple_name, SUM(p.amount) AS TotalRevenue
  FROM payments p
  JOIN tickets t ON p.ticket_id = t.ticket_id
  JOIN temple_details td ON td.temple_id = t.temple_id
  WHERE td.temple_id = temple
  GROUP BY td.temple_name;
END $$

CREATE FUNCTION TotalPayments() RETURNS INT
DETERMINISTIC
BEGIN
  DECLARE total INT;
  SELECT COUNT(payment_id) INTO total FROM payments;
  RETURN total;
END $$

DELIMITER ;

-- TASK 7: Triggers, Views and Exception Handling

CREATE VIEW payment_summary AS
SELECT td.temple_name, COUNT(p.payment_id) AS TotalPayments, SUM(p.amount) AS Revenue
FROM payments p
JOIN tickets t ON p.ticket_id = t.ticket_id
JOIN temple_details td ON td.temple_id = t.temple_id
GROUP BY td.temple_name;

DELIMITER $$
CREATE TRIGGER after_payment_insert
AFTER INSERT ON payments
FOR EACH ROW
BEGIN
  INSERT INTO admin (admin_name, email, password)
  VALUES ('System Log', CONCAT('log_', NOW()), 'auto');
END $$
DELIMITER ;

// TASK 8: MongoDB CRUD using Mongoose (Node.js)

db.users.insertOne({ user_name: "Rama", email: "rama@gmail.com", phone: "9876543210" });
db.temples.insertOne({ temple_name: "Tirupati", location: "Tirupati" });

// READ
db.temples.find();

// UPDATE
db.temples.updateOne({ temple_name: "Tirupati" }, { $set: { location: "Andhra Pradesh" } });

// DELETE
db.users.deleteOne({ user_name: "Rama" });


// TASK 9: Neo4j Graph Database CRUD

CREATE (u:User {name:'Rama'})
CREATE (t:Temple {name:'Tirupati Temple'})
CREATE (u)-[:BOOKED]->(t);

MATCH (u:User)-[:BOOKED]->(t:Temple) RETURN u.name, t.name;

MATCH (u:User {name:'Rama'}) DETACH DELETE u;

-- TASK 10: Normalization to BCNF

-- Step 1: Create unnormalized table
CREATE TABLE booking_raw (
  user_id INT,
  user_name VARCHAR(50),
  temple_id INT,
  temple_name VARCHAR(100),
  amount DECIMAL(8,2)
);

-- Step 2: 1NF
-- Separate repeating groups
CREATE TABLE users_nf (user_id INT PRIMARY KEY, user_name VARCHAR(50));
CREATE TABLE temples_nf (temple_id INT PRIMARY KEY, temple_name VARCHAR(100));
CREATE TABLE payments_nf (user_id INT, temple_id INT, amount DECIMAL(8,2));

-- Step 3: 2NF, 3NF, BCNF by separating full dependencies
ALTER TABLE payments_nf ADD FOREIGN KEY (user_id) REFERENCES users_nf(user_id);
ALTER TABLE payments_nf ADD FOREIGN KEY (temple_id) REFERENCES temples_nf(temple_id);

-- TASK 11: Menus, Forms and Reports
-- Conceptual Example for SQL Server / Oracle

-- Creating a simple view for report
CREATE VIEW ticket_report AS
SELECT u.user_name, td.temple_name, t.visit_date, p.amount, p.status
FROM users u
JOIN tickets t ON u.user_id = t.user_id
JOIN temple_details td ON t.temple_id = td.temple_id
JOIN payments p ON p.ticket_id = t.ticket_id;

-- Query for report
SELECT * FROM ticket_report ORDER BY visit_date;

-- Use Case: Army Supply Chain and Maintenance Cost Management

CREATE DATABASE army_supply;
USE army_supply;

CREATE TABLE equipment (
  eq_id INT AUTO_INCREMENT PRIMARY KEY,
  eq_name VARCHAR(100),
  category VARCHAR(50),
  mean_time_failure INT,
  maintenance_cost DECIMAL(10,2)
);

CREATE TABLE bases (
  base_id INT AUTO_INCREMENT PRIMARY KEY,
  base_name VARCHAR(100),
  location VARCHAR(100),
  climate VARCHAR(50)
);

CREATE TABLE deployment (
  dep_id INT AUTO_INCREMENT PRIMARY KEY,
  eq_id INT,
  base_id INT,
  deploy_date DATE,
  FOREIGN KEY (eq_id) REFERENCES equipment(eq_id),
  FOREIGN KEY (base_id) REFERENCES bases(base_id)
);

-- Insert Data
INSERT INTO equipment (eq_name, category, mean_time_failure, maintenance_cost)
VALUES ('Helicopter', 'Aircraft', 1500, 500000),
       ('Rifle', 'Weapon', 5000, 2000);

INSERT INTO bases (base_name, location, climate)
VALUES ('Northern Command', 'Jammu', 'Cold'),
       ('Southern Command', 'Chennai', 'Humid');

-- Analyze maintenance cost per location
SELECT b.location, AVG(e.maintenance_cost) AS AvgCost
FROM equipment e
JOIN deployment d ON e.eq_id = d.eq_id
JOIN bases b ON d.base_id = b.base_id
GROUP BY b.location;
