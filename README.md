# Library-Management-Systems

# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/najirh/Library-System-Management---P2/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
create database library; 
use library; 

create table branch (
	branch_id varchar(10) primary key, 	
	manager_id	varchar(10),
	branch_address	varchar(55),
	contact_no varchar(20)
); 

create table employees (
	emp_id	varchar(10) primary key,
    emp_name	varchar(25),
    position	varchar(15),
    salary	int,
    branch_id varchar(10),
    foreign key (branch_id) references branch(branch_id)
); 

create table books (
	isbn varchar(25) primary key,
    book_title	varchar(75),
    category	varchar(20),
    rental_price	float,
    status	varchar(15),
    author	varchar(35),
    publisher varchar(55)
    );
    
create table members (
	member_id	varchar(20) primary key,
    member_name	varchar(25),
    member_address varchar(15),
    reg_date date 
    ); 
    
    
create table issued_status (
	issued_id	varchar(10) primary key, 
	issued_member_id	varchar(20),
	issued_book_name	varchar(75),
	issued_date	date,
	issued_book_isbn	varchar(25),
	issued_emp_id varchar(10),
    foreign key (issued_member_id) references members (member_id), 
    foreign key (issued_book_isbn) references books(isbn),
    foreign key (issued_emp_id) references employees(emp_id)
        );


create table returned_status (
	return_id	varchar(10)  primary key, 
    issued_id	varchar(10),  
    return_book_name	varchar(25)  ,
    return_date	date,
    return_book_isbn varchar(20), 
    foreign key (issued_id) references issued_status(issued_id)
   ); 


### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.
```
**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
insert into books ( isbn , book_title, category, rental_price, status, author, publisher)
values 
('978-1-60129-456-2', 'To kill a Mockingbird' , ' classic', '6.00', 'yes', 'Harper Lee', 'J.B. Lippincot & co.' ); 
select * 
from books;
```
**Task 2: Update an Existing Member's Address**

```sql
select *
from members;

update members 
set member_address = '125 Main St' 
where member_id = 'C101';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
select *
from issued_status;

delete from issued_status 
where issued_id = 'IS106'; 

```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
select * from issued_status 
where issued_emp_id = 'E101'; 
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
select issued_emp_id, count(*) 
from issued_status 
group by issued_emp_id
having count(*) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
create table book_counts
as
select b.isbn , b.book_title,  count(status.issued_id) as number_of_times_issued
from books as b
inner join issued_status as status 
on b.isbn=status.issued_book_isbn 
group by b.isbn, b.book_title;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
select *
from books 
where category = 'Classic';

select *
from books 
where category = 'Fiction' ;
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
select b.category, sum(b.rental_price) as total_income
from books as b
inner join issued_status as status 
on b.isbn=status.issued_book_isbn 
group by b.category 
order by total_income desc;
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
values ('C120', 'Gapen' , '145 main st', '2025-04-01'), 
		('C121', 'tony' , '635 Park rd', '2025-03-29'); 
        
select *
from members 
where curdate() - reg_date <= 180;

```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
select e1.emp_id, e1.emp_name,
		m1.emp_name as manager_name,
        b1.branch_id as branch_id,
        b1.branch_address as branch_address,
        b1.contact_no 
from employees as e1
inner join branch as b1 
on e1.branch_id=b1.branch_id
inner join employees as m1 
on b1.manager_id=m1.emp_id;
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
create table expensive_books as
select *
from books
where rental_price > 5;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
select ist.issued_book_name
from issued_status as ist
left join returned_status as rst 
on ist.issued_id=rst.issued_id
where rst.return_id is null;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
select  m.member_id, 
m.member_name, 
ist.issued_book_name, 
ist.issued_date, 
datediff(curdate(), ist.issued_date) as days
from issued_status as ist

	left join returned_status as rst 
	on ist.issued_id=rst.issued_id
    
	left join members as m
	on ist.issued_member_id=m.member_id
    
where rst.return_id is null 
and datediff(curdate(), ist.issued_date) > 30
order by m.member_id;
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

select *
from issued_status 
where issued_book_isbn='978-0-451-52994-2';

select *
from books
where isbn = '978-0-451-52994-2';

update books 
set status = 'no'
where isbn = '978-0-451-52994-2';

select *
from returned_status
where issued_id='IS130'; -- We can see there is no record of this in returned_status table so this book is not returned 

-- If lets say this customer returned the book today, we can manually add the record into returned_status table and then change the status in books table as 'Yes' available

insert into returned_status (return_id,issued_id,return_date)
values ('RS125','IS130',curdate());

update books 
set status = 'yes'
where isbn = '978-0-451-52994-2';

-- Now using the SQL Trigger
drop trigger update_book_status_on_return;

DELIMITER $$

CREATE TRIGGER update_book_status_on_return
AFTER INSERT ON returned_status
FOR EACH ROW
BEGIN
  DECLARE v_isbn VARCHAR(25);

  SELECT issued_book_isbn
  INTO v_isbn
  FROM issued_status
  WHERE issued_id = NEW.issued_id;

  UPDATE books
  SET status = 'Yes'
  WHERE isbn = v_isbn;
END$$

DELIMITER ;


select *
from books
where isbn = '978-0-375-41398-8';

select *
from issued_status 
where issued_book_isbn = '978-0-375-41398-8';

select *
from returned_status
where return_id = 'IS134';

insert into returned_status (return_id,issued_id,return_date)
values ('RS127','IS134',curdate());
```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
create table branch_reports
as
select b.branch_id, 
		b.manager_id, 
        count(ist.issued_id) as number_of_book_issued,
        count(rst.return_id) as number_ofbooks_returned ,
        sum(bk.rental_price) as total_revenue 
        
from issued_status as ist
inner join 
employees as e
	on ist.issued_emp_id=e.emp_id
inner join 
branch as b
	on e.branch_id=b.branch_id
left join 
returned_status as rst
	on ist.issued_id=rst.issued_id 
join 
books as bk
	on ist.issued_book_isbn=bk.isbn
    
group by b.branch_id, 
		b.manager_id;
        

select *
from branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

create table active_members 
as 
select distinct issued_member_id, member_name
from issued_status as ist
join members as m 
on ist.issued_member_id=m.member_id
where timestampdiff(month, ist.issued_date, curdate()) <= 12;

select *
from active_members;


```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
select e.emp_id, e.emp_name, count(issued_book_isbn) as number_of_books_issued, b.branch_id
from issued_status as ist
join employees as e 
on ist.issued_emp_id=e.emp_id
join branch as b 
on e.branch_id=b.branch_id
group by e.emp_id, e.emp_name, b.branch_id
order by number_of_books_issued desc
limit 3; 
```

**Task 18: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.    


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

CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
-- all the variabable
    v_status VARCHAR(10);

BEGIN
-- all the code
    -- checking if book is available 'yes'
    SELECT 
        status 
        INTO
        v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN

        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES
        (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books
            SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        RAISE NOTICE 'Book records added successfully for book isbn : %', p_issued_book_isbn;


    ELSE
        RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
    END IF;
END;
$$

-- Testing The function
SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;

CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'

```



**Task 20: Create Table As Select (CTAS)**
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
    The number of overdue books.
    The total fines, with each day's fine calculated at $0.50.
    The number of books issued by each member.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines



## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

2. **Set Up the Database**: Execute the SQL scripts in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## Author - Zero Analyst

This project showcases SQL skills essential for database management and analysis. For more content on SQL and data analysis, connect with me through the following channels:
- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/abhishek-shakya-31147b346/)
- **Website_Profile**: [Visit my website] (https://aviskya1080.wixsite.com/my-site-5)

Thank you for your interest in this project!
