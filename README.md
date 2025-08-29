# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Database**: `LIBRARY_DB`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/najirh/Library-System-Management---P2/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD](https://github.com/BERLINSAMUELRAJ/LIBRARY_MANAGEMENT_SYSTEM/blob/main/LIBRARY%20MANAGEMENT%20SYSTEM%20SQL/Library_ER_Diagram.png)

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE LIBRARY_DB;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
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



-- Create table "IssueStatus"
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



-- Create table "ReturnStatus"
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

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books (ISBN, BOOK_TITLE, CATEGORY, RENTAL_PRICE, STATUS, AUTHOR, PUBLISHER)
VALUES ('978-1-60129-456-2', 'To Kill a Mockingbird','Classic', 6.00, 'yes', 
'Harper Lee', 'J.B. Lippincott & Co.')
```
**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '198 Berlin Samuel Raj St'
WHERE member_id = 'C119';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE issued_id= 'IS121'
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id= 'E101';
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
WITH ISSUED_CTE AS(
	SELECT issued_emp_id, COUNT(*) AS "COUNT OF BOOKS" FROM issued_status
	GROUP BY issued_emp_id
	HAVING COUNT(*)>1
	)
SELECT * FROM ISSUED_CTE
ORDER BY [COUNT OF BOOKS] DESC;
GO
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
SELECT 
    B.isbn, 
    B.book_title, 
    COUNT(IST.ISSUED_ID) AS issue_count  
INTO BOOK_ISSUED_COUNT  -- Creates and populates the table in one step
FROM books B 
LEFT JOIN issued_status IST ON B.isbn = IST.issued_book_isbn
GROUP BY B.isbn, B.book_title;

SELECT * FROM BOOK_ISSUED_COUNT;
GO
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category= 'Classic';
GO
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
WITH RENTAL_CTE AS(
	SELECT B.category, SUM(B.rental_price) AS "TOTAL RENTAL INCOME", COUNT(*) AS 
	"COUNT OF ISSUES" FROM books B JOIN issued_status IST
	ON B.isbn = IST.issued_book_isbn
	GROUP BY category
)
SELECT * FROM RENTAL_CTE
ORDER BY [TOTAL RENTAL INCOME] DESC;
GO
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
WITH "180 CTE" AS (
	SELECT member_id, member_name, reg_date , 
	DATEDIFF(DAY, reg_date, GETDATE()) AS "DAYS COUNT",
	CASE 
		WHEN DATEDIFF(DAY, reg_date, GETDATE())<=180 THEN 'WITHIN 180 DAYS'
		WHEN DATEDIFF(DAY, reg_date, GETDATE())>180 THEN 'NOT WITHIN 180 DAYS'
		END AS "STATUS"
	FROM members
)

SELECT * FROM "180 CTE";
GO
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
WITH BRANCH_CTE AS(
	SELECT E.emp_id, E.emp_name, manager_id,BR.branch_id, branch_address,contact_no
	FROM employees E JOIN branch BR
	ON E.branch_id = BR. branch_id
)
SELECT * FROM BRANCH_CTE;
GO
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
SELECT * FROM books
WHERE rental_price>7.00;
GO
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
WITH BOOK_RETURNED_STATUS AS (
	SELECT M.member_id, M.member_name, IST.issued_book_name,IST.issued_date,
	DATEDIFF(DAY, IST.issued_date, RS.return_date) AS "DAYS OVERDUE",
	CASE 
		WHEN DATEDIFF(DAY, IST.issued_date, RS.return_date)>30 THEN 'OVERDUE'
		WHEN DATEDIFF(DAY, IST.issued_date, RS.return_date)<30 THEN 'NO ISSUES'
		END AS "BOOK RETURN STATUS"
	FROM issued_status IST JOIN members M
	ON M.member_id = IST.issued_member_id JOIN return_status RS
	ON IST.issued_id = RS.issued_id
)

SELECT * FROM BOOK_RETURNED_STATUS
ORDER BY [DAYS OVERDUE] DESC;
GO
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

CREATE PROCEDURE add_return_records
    @p_return_id VARCHAR(10),
    @p_issued_id VARCHAR(10),
    @p_book_quality VARCHAR(10)
AS
BEGIN
    DECLARE @v_isbn VARCHAR(50);
    DECLARE @v_book_name VARCHAR(80);

    -- Insert into return_status
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES (@p_return_id, @p_issued_id, CAST(GETDATE() AS DATE), @p_book_quality);

    -- Get isbn and book name from issued_status
    SELECT 
        @v_isbn = issued_book_isbn,
        @v_book_name = issued_book_name
    FROM issued_status
    WHERE issued_id = @p_issued_id;

    -- Update books table
    UPDATE books
    SET status = 'yes'
    WHERE isbn = @v_isbn;

    -- Print a message
    PRINT 'Thank you for returning the book: ' + @v_book_name;
END;

-- Testing FUNCTION add_return_records --

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * FROM return_status
WHERE issued_id = 'IS135';

SELECT * FROM return_status
WHERE issued_id = 'IS140';

SELECT * FROM issued_status
WHERE issued_id = 'IS140';

SELECT * FROM books
WHERE isbn = '978-0-330-25864-8';

-- Executing functions and checking --
EXEC add_return_records 
    @p_return_id = 'RS138',
    @p_issued_id = 'IS135',
    @p_book_quality = 'Good';

SELECT *
FROM return_status
WHERE return_id = 'RS138';

EXEC add_return_records 
    @p_return_id = 'RS148',
    @p_issued_id = 'IS140',
    @p_book_quality = 'Good';

SELECT *
FROM return_status
WHERE return_id = 'RS148';

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
SELECT 
    br.branch_id, 
    br.manager_id,
    COUNT(DISTINCT iu.issued_id) AS [TOTAL BOOKS ISSUED], 
    COUNT(DISTINCT r.return_id) AS [TOTAL BOOKS RETURNED], 
    SUM(bk.rental_price) AS [Total Book Revenue]
INTO branch_reports   -- creates new table + inserts data
FROM branch br
INNER JOIN employees e 
    ON e.branch_id = br.branch_id  
INNER JOIN issued_status iu 
    ON iu.issued_emp_id = e.emp_id
INNER JOIN books bk 
    ON bk.isbn = iu.issued_book_isbn 
LEFT JOIN return_status r 
    ON iu.issued_id = r.issued_id
GROUP BY br.branch_id, br.manager_id, br.branch_address, br.contact_no;

-- Running the new table created --
SELECT * FROM branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

SELECT 
    m.member_id, 
    m.member_name, 
    i.issued_id,
    i.issued_book_isbn,
    i.issued_book_name, 
    i.issued_date
INTO active_members
FROM members m
INNER JOIN issued_status i 
    ON m.member_id = i.issued_member_id
WHERE i.issued_date >= (
    SELECT DATEADD(MONTH, -2, MAX(issued_date)) 
    FROM issued_status
);

-- Querying the active members table --
SELECT * FROM active_members;

```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
WITH EmpIssues AS (
    SELECT 
        DENSE_RANK() OVER (ORDER BY COUNT(i.issued_id) DESC) AS RANK_NO,
        e.emp_name, 
        COUNT(i.issued_id) AS number_of_books_processed, 
        b.branch_id
    FROM branch b
    INNER JOIN employees e 
        ON e.branch_id = b.branch_id
    INNER JOIN issued_status i 
        ON i.issued_emp_id = e.emp_id
    GROUP BY e.emp_name, b.branch_id
)
SELECT *
FROM EmpIssues
WHERE RANK_NO <= 3
ORDER BY number_of_books_processed DESC;
```  

**Task 19: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql
CREATE PROCEDURE manage_book_status
    @book_id VARCHAR(50)
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @current_status VARCHAR(10);

    -- Check the current status of the book
    SELECT @current_status = status
    FROM books
    WHERE isbn = @book_id;

    -- If book does not exist
    IF @current_status IS NULL
    BEGIN
        PRINT 'Error: Book with given ID not found.';
        RETURN;
    END;

    -- If book is available
    IF @current_status = 'yes'
    BEGIN
        UPDATE books
        SET status = 'no'
        WHERE isbn = @book_id;

        PRINT 'Book has been issued successfully.';
    END
    ELSE
    BEGIN
        PRINT 'Error: Book is currently not available.';
    END;
END;

-- Case 1: Book is available ("yes")
EXEC manage_book_status @book_id = '978-0-553-29698-2';

-- Case 2: Book is not available ("no")
EXEC manage_book_status @book_id = '978-0-375-41398-8';
```

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
