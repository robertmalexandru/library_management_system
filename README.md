# Library Management System

This project implements a **Library Management System** using SQL. It involves database design, table creation, and performing CRUD operations alongside advanced SQL queries. The objective is to showcase **proficiency in database management, data analysis, and query optimization**.

![Library Project](library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create tables for branches, employees, members, books, issued books, and returned books.
2. **CRUD Operations**: Implement Create, Read, Update, and Delete operations.
3. **CTAS (Create Table As Select)**: Use CTAS to create new tables based on queries.
4. **Advanced SQL Queries**: Execute complex queries to retrieve and analyze data.

### 1. Database Setup

![ERD](library_erd.png)

#### **Schema Overview**
- **`library_db`**: Main database.
- **Tables**: `branch`, `employees`, `members`, `books`, `issued_status`, `return_status`.

```sql
CREATE DATABASE library_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch(branch_id)
);

DROP TABLE IF EXISTS members;
CREATE TABLE members
(
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

DROP TABLE IF EXISTS books;
CREATE TABLE books
(
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(80),
    category VARCHAR(30),
    rental_price DECIMAL(10,2),
    status VARCHAR(10),
    author VARCHAR(30),
    publisher VARCHAR(30)
);

DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member_id VARCHAR(30),
    issued_book_name VARCHAR(80),
    issued_date DATE,
    issued_book_isbn VARCHAR(50),
    issued_emp_id VARCHAR(10),
    FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
    FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
    FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);

DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```

### 2. CRUD Operations

#### **Insert a New Book**
```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES ('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

#### **Update Member Address**
```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

#### **Delete Issued Book Record**
```sql
DELETE FROM issued_status
WHERE issued_id = 'IS121';
```

#### **Retrieve All Books Issued by an Employee**
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
```

### 3. CTAS (Create Table As Select)

#### **Create Summary Table for Book Issues**
```sql
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status AS ist
JOIN books AS b ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
```

### 4. Data Analysis & Queries

#### **Retrieve Books in a Specific Category**
```sql
SELECT * FROM books WHERE category = 'Classic';
```

#### **Calculate Total Rental Income by Category**
```sql
SELECT b.category, SUM(b.rental_price), COUNT(*)
FROM issued_status AS ist
JOIN books AS b ON b.isbn = ist.issued_book_isbn
GROUP BY b.category;
```

#### **Find Employees Who Processed the Most Books**
```sql
SELECT e.emp_name, COUNT(ist.issued_id) AS no_books_processed
FROM issued_status AS ist
JOIN employees AS e ON e.emp_id = ist.issued_emp_id
GROUP BY e.emp_name
ORDER BY no_books_processed DESC
LIMIT 3;
```

#### **Identify Overdue Books**
```sql
SELECT ist.issued_member_id, m.member_name, bk.book_title, ist.issued_date,
       CURRENT_DATE - ist.issued_date AS overdue_days
FROM issued_status AS ist
JOIN members AS m ON m.member_id = ist.issued_member_id
JOIN books AS bk ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL AND (CURRENT_DATE - ist.issued_date) > 30;
```

#### **Stored Procedure: Issue a Book**
```sql
CREATE OR REPLACE PROCEDURE issue_book(
    p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), 
    p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_status VARCHAR(10);
BEGIN
    SELECT status INTO v_status FROM books WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN
        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books SET status = 'no' WHERE isbn = p_issued_book_isbn;
        RAISE NOTICE 'Book successfully issued: %', p_issued_book_isbn;
    ELSE
        RAISE NOTICE 'Book unavailable: %', p_issued_book_isbn;
    END IF;
END;
$$;
```

## Reports

- **Data Analysis**: Includes member registration trends, book rentals, and employee performance.
- **Summary Reports**: 
  - Books issued per branch
  - High-demand book categories
  - Employee performance metrics
