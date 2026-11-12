# DBMS Lab – Oracle SQL*Plus Practice Programs

Oracle SQL / SQL*Plus version of the DBMS lab (Part A and Part B). Every program has the SQL, a "Tables used" block, a short memory note, and sample output for the queries.

## Contents

- [What changed from the MySQL version](#what-changed-from-the-mysql-version)
- [How to run in SQL*Plus](#how-to-run-in-sqlplus)
- [Part A – Company schema](#part-a--company-schema)
  - [Program 2](#part-a--program-2)
  - [Program 3](#part-a--program-3)
  - [Program 4](#part-a--program-4)
  - [Program 5](#part-a--program-5)
  - [Program 6](#part-a--program-6)
  - [Program 7](#part-a--program-7)
  - [Program 8](#part-a--program-8)
  - [Program 9](#part-a--program-9)
  - [Program 10](#part-a--program-10)
- [Part B – Library and Student schemas](#part-b--library-and-student-schemas)
  - [Program 1](#part-b--program-1)
  - [Program 2](#part-b--program-2)
  - [Program 3](#part-b--program-3)
  - [Program 4](#part-b--program-4)
  - [Program 5](#part-b--program-5)
  - [Program 6](#part-b--program-6)
  - [Program 7](#part-b--program-7)
  - [Program 8](#part-b--program-8)
  - [Program 9](#part-b--program-9)
  - [Program 10](#part-b--program-10)

## What changed from the MySQL version

| MySQL | Oracle / SQL*Plus |
|---|---|
| `CREATE DATABASE`, `USE`, `SHOW DATABASES` | A user is the schema: `CREATE USER`, `GRANT`, `CONNECT`, `SHOW USER` |
| `SHOW TABLES;` | `SELECT table_name FROM user_tables;` |
| `INT`, `VARCHAR`, `DECIMAL` | `NUMBER`, `VARCHAR2`, `NUMBER(p,s)` |
| `'1990-01-10'` | `DATE '1990-01-10'` |
| One `INSERT` with many rows | One `INSERT` per row |
| `YEAR(date)` | `EXTRACT(YEAR FROM date)` |
| `ADD col type`, `MODIFY col type` | `ADD (col type)`, `MODIFY (col type)` |
| `RENAME TABLE a TO b` | `ALTER TABLE a RENAME TO b` |
| `START TRANSACTION` | Not needed. DML starts a transaction automatically |
| Auto-commit after each statement | Run `COMMIT;` yourself after INSERT/UPDATE/DELETE |
| `mysqldump` / `mysql` | `expdp` / `impdp` (Data Pump) |
| `LIKE` ignores case | `LIKE` is case-sensitive, so `UPPER(name) LIKE 'S%'` is used |

## How to run in SQL*Plus

1. Open Command Prompt and log in as a DBA user:
   ```text
   sqlplus system/<password>@localhost:1521/XEPDB1
   ```
   Use your own password and service name. If `CREATE USER` gives ORA-65096, you are in the container database. Connect to the pluggable database (for example `XEPDB1`) instead.
2. Set up the session. Do this every time you log in:
   ```sql
   SET LINESIZE 200
   SET PAGESIZE 50
   ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY-MM-DD';
   ```
3. Run each part in order, top to bottom, because some programs change data.
4. Part A uses the `company` user. Part B uses the `library` user (Programs 1–5) and the `students` user (Programs 6–10).
5. Useful commands: `DESC EMP` (table structure), `SPOOL output.txt` / `SPOOL OFF` (save output to a file), `@script.sql` (run a script), `EXIT`.
6. If you log in as the SCOTT demo user, `EMP` and `DEPT` already exist. Use the `company` user instead to avoid a clash.

---

## Part A – Company schema

*Tables: EMP, DEPT, PROJECT, WORKS_ON*

### Setup and Required Company Tables

Run the setup once, then create the four tables. Use them for all Programs 2–10. No additional tables or attributes are required.

#### Setup (create the schema)

```sql
-- Run as SYSTEM (or any DBA user)
CREATE USER company IDENTIFIED BY company123;
GRANT CONNECT, RESOURCE, CREATE VIEW, UNLIMITED TABLESPACE TO company;

-- Switch to the new schema
CONNECT company/company123
```

#### Create tables

```sql
CREATE TABLE DEPT (
    DNO NUMBER(3) PRIMARY KEY,
    DNAME VARCHAR2(30),
    DLOCATION VARCHAR2(30)
);

CREATE TABLE EMP (
    ENO NUMBER(5) PRIMARY KEY,
    ENAME VARCHAR2(30),
    EBDATE DATE,
    ADDRESS VARCHAR2(100),
    GENDER CHAR(1),
    SALARY NUMBER(10,2),
    DEPTNO NUMBER(3),
    FOREIGN KEY (DEPTNO) REFERENCES DEPT(DNO)
);

CREATE TABLE PROJECT (
    PNO NUMBER(3) PRIMARY KEY,
    PNAME VARCHAR2(30),
    DNUM NUMBER(3),
    FOREIGN KEY (DNUM) REFERENCES DEPT(DNO)
);

CREATE TABLE WORKS_ON (
    ENO NUMBER(5),
    PNUM NUMBER(3),
    HOURS NUMBER(5,2),
    PRIMARY KEY (ENO, PNUM),
    FOREIGN KEY (ENO) REFERENCES EMP(ENO),
    FOREIGN KEY (PNUM) REFERENCES PROJECT(PNO)
);
```

### Part A – Program 2

> **Perform the following:**  
> **a. Viewing all databases, Creating a Database, Viewing all Tables in a Database,**  
> **Creating Tables (With and Without Constraints), Inserting/Updating/Deleting**  
> **Records in a Table, Saving (Commit) and Undoing (rollback)**  

#### View databases (Oracle: database name and users/schemas)

```sql
SELECT name FROM v$database;
SELECT username FROM all_users ORDER BY username;
SHOW USER;
```

#### Create database (Oracle uses a user/schema)

```sql
-- Already done in 'Setup' above: CREATE USER company ... then CONNECT company/company123
SHOW USER;
```

#### View tables

```sql
SELECT table_name FROM user_tables;
```

#### Create table without constraints

```sql
CREATE TABLE TEST (
    ID NUMBER,
    NAME VARCHAR2(30)
);
```

#### Insert records

```sql
INSERT INTO DEPT VALUES (1,'RESEARCH','BANGALORE');
INSERT INTO DEPT VALUES (2,'ACCOUNTS','MYSORE');
INSERT INTO DEPT VALUES (5,'ADMIN','BANGALORE');

INSERT INTO EMP VALUES (1001,'ANIL',DATE '1990-01-10','BANGALORE','M',45000,1);
INSERT INTO EMP VALUES (1002,'SINDHU',DATE '1991-05-20','MYSORE','F',50000,2);
INSERT INTO EMP VALUES (1003,'RAJ',DATE '1990-11-15','BANGALORE','M',35000,1);
INSERT INTO EMP VALUES (1004,'KAVYA',DATE '1992-03-12','MYSORE','F',55000,2);
INSERT INTO EMP VALUES (1005,'ARUN',DATE '1989-08-10','BANGALORE','M',30000,5);
INSERT INTO EMP VALUES (1006,'SNEHA',DATE '1990-12-05','BANGALORE','F',45000,5);

INSERT INTO PROJECT VALUES (10,'ERP',1);
INSERT INTO PROJECT VALUES (20,'BANKING',2);
INSERT INTO PROJECT VALUES (30,'PAYROLL',5);

INSERT INTO WORKS_ON VALUES (1001,10,5);
INSERT INTO WORKS_ON VALUES (1002,20,6);
INSERT INTO WORKS_ON VALUES (1003,10,4);
INSERT INTO WORKS_ON VALUES (1003,20,3);
INSERT INTO WORKS_ON VALUES (1004,20,7);
INSERT INTO WORKS_ON VALUES (1005,30,5);
INSERT INTO WORKS_ON VALUES (1006,30,6);

COMMIT;
```

#### Update

```sql
UPDATE EMP SET SALARY = 50000 WHERE ENO = 1001;
```

#### Delete

```sql
INSERT INTO EMP VALUES (1007,'TEMP',DATE '1993-01-01','BANGALORE','M',20000,1);
DELETE FROM EMP WHERE ENO = 1007;
```

#### Commit

```sql
COMMIT;
```

#### Rollback

```sql
UPDATE EMP SET SALARY = 60000 WHERE ENO = 1002;
ROLLBACK;
```

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### Memory

COMMIT = save. ROLLBACK = undo. In Oracle, INSERT/UPDATE/DELETE are not saved until COMMIT. CREATE/ALTER/DROP commit automatically.

### Part A – Program 3

> **Perform the following:**  
> **a. Altering a Table, Dropping/Truncating/Renaming Tables, Backing up / Restoring a**  
> **Database.**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### Add column

```sql
ALTER TABLE EMP ADD (EMAIL VARCHAR2(50));
```

#### Modify column

```sql
ALTER TABLE EMP MODIFY (EMAIL VARCHAR2(100));
```

#### Rename column

```sql
ALTER TABLE EMP RENAME COLUMN EMAIL TO EMAIL_ID;
```

#### Rename table

```sql
ALTER TABLE EMP RENAME TO EMPLOYEE;
ALTER TABLE EMPLOYEE RENAME TO EMP;
```

#### Truncate

```sql
TRUNCATE TABLE TEST;
```

#### Drop

```sql
DROP TABLE TEST;
```

#### Backup – one-time setup (run as SYSTEM)

```sql
-- Create the folder first (for example C:\backup), then:
CREATE OR REPLACE DIRECTORY backup_dir AS 'C:\backup';
GRANT READ, WRITE ON DIRECTORY backup_dir TO company;
```

#### Backup – Command Prompt (not inside SQL*Plus)

```text
expdp company/company123@localhost:1521/XEPDB1 schemas=COMPANY directory=BACKUP_DIR dumpfile=company_backup.dmp logfile=company_exp.log
```

#### Restore – Command Prompt

```text
impdp company/company123@localhost:1521/XEPDB1 schemas=COMPANY directory=BACKUP_DIR dumpfile=company_backup.dmp logfile=company_imp.log table_exists_action=replace
```

#### Memory

ALTER = change. TRUNCATE = remove all rows. DROP = remove table. expdp = backup, impdp = restore.

### Part A – Program 4

> **For a given set of relation schemes, create tables and perform the following Simple Queries,**  
> **Simple Queries with Aggregate functions, Queries with Aggregate functions (group by and having clause).**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### Simple queries

```sql
SELECT * FROM EMP;
SELECT ENAME, SALARY FROM EMP;
SELECT * FROM EMP WHERE SALARY > 40000;
SELECT * FROM EMP WHERE DEPTNO = 1;
SELECT * FROM EMP ORDER BY SALARY;
```

#### Aggregate functions

```sql
SELECT COUNT(*) FROM EMP;
SELECT SUM(SALARY) FROM EMP;
SELECT AVG(SALARY) FROM EMP;
SELECT MAX(SALARY) FROM EMP;
SELECT MIN(SALARY) FROM EMP;
```

**Output (sample data):**

```text
COUNT(*)
--------
6       

SUM(SALARY)
-----------
260000     

AVG(SALARY)
-----------
43333.3    

MAX(SALARY)
-----------
55000      

MIN(SALARY)
-----------
30000      
```

#### GROUP BY

```sql
SELECT DEPTNO, COUNT(*) FROM EMP GROUP BY DEPTNO;
```

**Output (sample data):**

```text
DEPTNO COUNT(*)
------ --------
1      2       
2      2       
5      2       
```

#### GROUP BY + HAVING

```sql
SELECT DEPTNO, COUNT(*)
FROM EMP
GROUP BY DEPTNO
HAVING COUNT(*) >= 2;
```

**Output (sample data):**

```text
DEPTNO COUNT(*)
------ --------
1      2       
2      2       
5      2       
```

#### Memory

GROUP BY makes groups. HAVING filters groups.

### Part A – Program 5

> **Execute the fallowing queries**  
> **a. How the resulting salaries if every employee working on the ‘Research’ Departments is**  
> **given a 10% raise.**  
> **b. Find the sum of the salaries of all employees of the ‘Accounts’ department, as well as the**  
> **maximum salary, the minimum salary, and the average salary in this department**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### 5(a)

```sql
SELECT
    E.ENAME,
    E.SALARY,
    E.SALARY * 1.10 AS NEW_SALARY
FROM EMP E
JOIN DEPT D ON E.DEPTNO = D.DNO
WHERE D.DNAME = 'RESEARCH';
```

**Output (sample data):**

```text
ENAME SALARY NEW_SALARY
----- ------ ----------
ANIL  45000  49500     
RAJ   35000  38500     
```

#### 5(b)

```sql
SELECT
    SUM(E.SALARY) AS TOTAL_SALARY,
    MAX(E.SALARY) AS MAX_SALARY,
    MIN(E.SALARY) AS MIN_SALARY,
    AVG(E.SALARY) AS AVG_SALARY
FROM EMP E
JOIN DEPT D ON E.DEPTNO = D.DNO
WHERE D.DNAME = 'ACCOUNTS';
```

**Output (sample data):**

```text
TOTAL_SALARY MAX_SALARY MIN_SALARY AVG_SALARY
------------ ---------- ---------- ----------
105000       55000      50000      52500     
```

#### Memory

JOIN connects EMP and DEPT using E.DEPTNO = D.DNO. Oracle compares text with case, so write 'RESEARCH' exactly as stored.

### Part A – Program 6

> **Execute the fallowing queries**  
> **a. Retrieve the name of each employee Controlled by department number 5 (use EXISTS**  
> **operator).**  
> **b. Retrieve the name of each dept and number of employees working in each department which**  
> **has at least 2 employees**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### 6(a) – EXISTS

```sql
SELECT E.ENAME
FROM EMP E
WHERE EXISTS (
    SELECT 1
    FROM DEPT D
    WHERE D.DNO = E.DEPTNO
    AND D.DNO = 5
);
```

**Output (sample data):**

```text
ENAME
-----
ARUN 
SNEHA
```

#### 6(b)

```sql
SELECT D.DNAME, COUNT(E.ENO) AS EMPLOYEE_COUNT
FROM DEPT D
JOIN EMP E ON D.DNO = E.DEPTNO
GROUP BY D.DNO, D.DNAME
HAVING COUNT(E.ENO) >= 2;
```

**Output (sample data):**

```text
DNAME    EMPLOYEE_COUNT
-------- --------------
RESEARCH 2             
ACCOUNTS 2             
ADMIN    2             
```

#### Memory

EXISTS checks whether a matching row exists.

### Part A – Program 7

> **Execute the fallowing queries**  
> **a. For each project, retrieve the project number, the project name, and the number of employee**  
> **who work on that project.(use GROUP BY)**  
> **b. Retrieve the name of employees who born in the year 1990’s**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### 7(a)

```sql
SELECT
    P.PNO,
    P.PNAME,
    COUNT(W.ENO) AS EMPLOYEE_COUNT
FROM PROJECT P
JOIN WORKS_ON W ON P.PNO = W.PNUM
GROUP BY P.PNO, P.PNAME;
```

**Output (sample data):**

```text
PNO PNAME   EMPLOYEE_COUNT
--- ------- --------------
10  ERP     2             
20  BANKING 3             
30  PAYROLL 2             
```

#### 7(b)

```sql
SELECT ENAME, EBDATE
FROM EMP
WHERE EXTRACT(YEAR FROM EBDATE) BETWEEN 1990 AND 1999;
```

**Output (sample data):**

```text
ENAME  EBDATE    
------ ----------
ANIL   1990-01-10
SINDHU 1991-05-20
RAJ    1990-11-15
KAVYA  1992-03-12
SNEHA  1990-12-05
```

#### Memory

Oracle has no YEAR() function. Use EXTRACT(YEAR FROM date). 1990s = 1990 to 1999.

### Part A – Program 8

> **For each department that has more than five employees, retrieve the department number and number of employees who are making salary more than 40000.**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### SQL

```sql
SELECT
    D.DNO,
    COUNT(*) AS EMPLOYEE_COUNT
FROM EMP E
JOIN DEPT D ON E.DEPTNO = D.DNO
WHERE E.SALARY > 40000
AND E.DEPTNO IN (
    SELECT DEPTNO
    FROM EMP
    GROUP BY DEPTNO
    HAVING COUNT(*) > 5
)
GROUP BY D.DNO;
```

**Output (sample data):**

```text
no rows selected
```

#### Practice note

With the small sample data, this returns no rows because no department has more than 5 employees. Add more employees to a department to see a result.

#### Memory

First find departments having > 5 employees, then count employees earning > 40000.

### Part A – Program 9

> **For each project on which more than two employees work, retrieve the project number, project name**  
> **and the number of employees who work on that project.**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### SQL

```sql
SELECT
    P.PNO,
    P.PNAME,
    COUNT(W.ENO) AS EMPLOYEE_COUNT
FROM PROJECT P
JOIN WORKS_ON W ON P.PNO = W.PNUM
GROUP BY P.PNO, P.PNAME
HAVING COUNT(W.ENO) > 2;
```

**Output (sample data):**

```text
PNO PNAME   EMPLOYEE_COUNT
--- ------- --------------
20  BANKING 3             
```

#### Memory

HAVING COUNT(...) > 2 means projects with more than 2 employees.

### Part A – Program 10

> **For a given set of relation tables perform the following**  
> **a. Creating Views (with and without check option), Dropping views, Selecting from a**  
> **view**  

#### Tables used (DEPT, EMP, PROJECT, WORKS_ON)

```sql
SELECT * FROM DEPT;
SELECT * FROM EMP;
SELECT * FROM PROJECT;
SELECT * FROM WORKS_ON;
```

#### Create view – without CHECK OPTION

```sql
CREATE VIEW EMP_DEPT AS
SELECT E.ENO, E.ENAME, E.SALARY, D.DNAME
FROM EMP E
JOIN DEPT D ON E.DEPTNO = D.DNO;
```

#### Select

```sql
SELECT * FROM EMP_DEPT;
```

#### Drop

```sql
DROP VIEW EMP_DEPT;
```

#### Create view – WITH CHECK OPTION

```sql
CREATE VIEW EMP_VIEW AS
SELECT ENO, ENAME, SALARY
FROM EMP
WHERE SALARY <= 50000
WITH CHECK OPTION;
```

#### Select

```sql
SELECT * FROM EMP_VIEW;
```

#### Valid insert

```sql
INSERT INTO EMP_VIEW VALUES (1010,'MANOJ',45000);
```

#### Invalid insert (gives an error)

```sql
INSERT INTO EMP_VIEW VALUES (1011,'RAHUL',60000);
-- ORA-01402: view WITH CHECK OPTION where-clause violation
```

#### Drop

```sql
DROP VIEW EMP_VIEW;
```

#### Memory

VIEW = virtual table. CHECK OPTION = inserted/updated data must satisfy the view condition. CREATE VIEW needs the CREATE VIEW privilege (given in Setup).

---

## Part B – Library and Student schemas

*Programs 1–5: BRANCH, STUDENT, BOOK, AUTHOR, BORROW. Programs 6–10: STUDENT (marks and GPA).*

### Schemas and Keys

Programs 1–5 use the `library` user and Programs 6–10 use the `students` user. Both have a table called STUDENT, so they are kept in separate users. Run Program 1 first, then Program 6, and run the programs in order because Programs 9 and 10 change data.

```text
BRANCH (Branchid PK, Branchname, HOD)
STUDENT (USN PK, Name, Address, Branchid FK, sem)
BOOK (Bookid PK, Bookname, Authorid FK, Publisher, Branchid FK)
AUTHOR (Authorid PK, Authorname, Country, age)
BORROW (USN FK, Bookid FK, Borrowed_Date)  -- PK (USN, Bookid)
```

### Part B – Program 1

> **Perform the following:**  
> **a. Viewing all databases, Creating a Database, Viewing all Tables in a Database,**  
> **Creating Tables (With and Without Constraints), Inserting/Updating/Deleting**  
> **Records in a Table, Saving (Commit) and Undoing (rollback)**  

#### View databases (Oracle: database name and users/schemas)

```sql
SELECT name FROM v$database;
SELECT username FROM all_users ORDER BY username;
SHOW USER;
```

#### Create database (Oracle uses a user/schema)

```sql
-- Run as SYSTEM (or any DBA user)
CREATE USER library IDENTIFIED BY library123;
GRANT CONNECT, RESOURCE, CREATE VIEW, UNLIMITED TABLESPACE TO library;

-- Switch to the new schema
CONNECT library/library123
```

#### Create tables (with constraints)

```sql
CREATE TABLE BRANCH (
    Branchid NUMBER(3) PRIMARY KEY,
    Branchname VARCHAR2(30) NOT NULL,
    HOD VARCHAR2(30)
);

CREATE TABLE AUTHOR (
    Authorid NUMBER(4) PRIMARY KEY,
    Authorname VARCHAR2(40) NOT NULL,
    Country VARCHAR2(30),
    age NUMBER(3)
);

CREATE TABLE STUDENT (
    USN VARCHAR2(12) PRIMARY KEY,
    Name VARCHAR2(40) NOT NULL,
    Address VARCHAR2(100),
    Branchid NUMBER(3),
    sem NUMBER(2),
    FOREIGN KEY (Branchid) REFERENCES BRANCH(Branchid)
);

CREATE TABLE BOOK (
    Bookid NUMBER(4) PRIMARY KEY,
    Bookname VARCHAR2(60) NOT NULL,
    Authorid NUMBER(4),
    Publisher VARCHAR2(40),
    Branchid NUMBER(3),
    FOREIGN KEY (Authorid) REFERENCES AUTHOR(Authorid),
    FOREIGN KEY (Branchid) REFERENCES BRANCH(Branchid)
);

CREATE TABLE BORROW (
    USN VARCHAR2(12),
    Bookid NUMBER(4),
    Borrowed_Date DATE,
    PRIMARY KEY (USN, Bookid),
    FOREIGN KEY (USN) REFERENCES STUDENT(USN),
    FOREIGN KEY (Bookid) REFERENCES BOOK(Bookid)
);
```

#### View tables

```sql
SELECT table_name FROM user_tables;
```

#### Create table without constraints

```sql
CREATE TABLE TEST (
    ID NUMBER,
    NAME VARCHAR2(30)
);
```

#### Insert records

```sql
INSERT INTO BRANCH VALUES (1,'MCA','Dr. Rao');
INSERT INTO BRANCH VALUES (2,'MBA','Dr. Meera');
INSERT INTO BRANCH VALUES (3,'BCA','Dr. Hegde');

INSERT INTO AUTHOR VALUES (101,'Korth','USA',60);
INSERT INTO AUTHOR VALUES (102,'Navathe','USA',58);
INSERT INTO AUTHOR VALUES (103,'Elmasri','USA',62);
INSERT INTO AUTHOR VALUES (104,'Kanetkar','India',55);
INSERT INTO AUTHOR VALUES (105,'Balagurusamy','India',70);

INSERT INTO STUDENT VALUES ('1SI24MC001','Anil','Bangalore',1,2);
INSERT INTO STUDENT VALUES ('1SI24MC002','Sindhu','Mysore',1,2);
INSERT INTO STUDENT VALUES ('1SI24MC003','Raj','Bangalore',1,2);
INSERT INTO STUDENT VALUES ('1SI24MC004','Kavya','Mysore',1,2);
INSERT INTO STUDENT VALUES ('1SI24MC005','Arun','Bangalore',1,1);
INSERT INTO STUDENT VALUES ('1SI24MB001','Sneha','Bangalore',2,2);

INSERT INTO BOOK VALUES (201,'DBMS Concepts',101,'McGraw Hill',1);
INSERT INTO BOOK VALUES (202,'Fundamentals of DBMS',103,'Pearson',1);
INSERT INTO BOOK VALUES (203,'Let Us C',104,'BPB',1);
INSERT INTO BOOK VALUES (204,'Java Programming',105,'McGraw Hill',1);
INSERT INTO BOOK VALUES (205,'Operating Systems',101,'McGraw Hill',1);
INSERT INTO BOOK VALUES (206,'Principles of Management',105,'Pearson',2);

INSERT INTO BORROW VALUES ('1SI24MC001',201,DATE '2026-08-01');
INSERT INTO BORROW VALUES ('1SI24MC001',203,DATE '2026-08-05');
INSERT INTO BORROW VALUES ('1SI24MC001',204,DATE '2026-08-10');
INSERT INTO BORROW VALUES ('1SI24MC002',201,DATE '2026-08-02');
INSERT INTO BORROW VALUES ('1SI24MC002',205,DATE '2026-08-12');
INSERT INTO BORROW VALUES ('1SI24MC003',202,DATE '2026-08-03');
INSERT INTO BORROW VALUES ('1SI24MC003',203,DATE '2026-08-15');

COMMIT;
```

#### Update

```sql
UPDATE STUDENT SET Address = 'Mysore' WHERE USN = '1SI24MC005';
UPDATE STUDENT SET Address = 'Bangalore' WHERE USN = '1SI24MC005';
```

#### Delete

```sql
INSERT INTO STUDENT VALUES ('1SI24MC099','TEMP','Bangalore',1,2);
DELETE FROM STUDENT WHERE USN = '1SI24MC099';
```

#### Commit

```sql
COMMIT;
```

#### Rollback

```sql
UPDATE STUDENT SET sem = 3 WHERE USN = '1SI24MC001';
ROLLBACK;
```

#### Tables used (BRANCH, STUDENT, BOOK, AUTHOR, BORROW)

```sql
SELECT * FROM BRANCH;
SELECT * FROM STUDENT;
SELECT * FROM BOOK;
SELECT * FROM AUTHOR;
SELECT * FROM BORROW;
```

#### Memory

COMMIT = save. ROLLBACK = undo. Create parent tables first (BRANCH, AUTHOR), then STUDENT, BOOK, BORROW. CREATE statements commit automatically.

### Part B – Program 2

> **Execute the following queries:**  
> **a. List the details of Students who are all studying in 2nd sem MCA.**  
> **b. List the students who are not borrowed any books.**  

#### Tables used (BRANCH, STUDENT, BOOK, AUTHOR, BORROW)

```sql
SELECT * FROM BRANCH;
SELECT * FROM STUDENT;
SELECT * FROM BOOK;
SELECT * FROM AUTHOR;
SELECT * FROM BORROW;
```

#### 2(a) Students studying in 2nd sem MCA

```sql
SELECT S.*
FROM STUDENT S
JOIN BRANCH B ON S.Branchid = B.Branchid
WHERE B.Branchname = 'MCA' AND S.sem = 2;
```

**Output (sample data):**

```text
USN        NAME   ADDRESS   BRANCHID SEM
---------- ------ --------- -------- ---
1SI24MC001 Anil   Bangalore 1        2  
1SI24MC002 Sindhu Mysore    1        2  
1SI24MC003 Raj    Bangalore 1        2  
1SI24MC004 Kavya  Mysore    1        2  
```

#### 2(b) Students who have not borrowed any books

```sql
SELECT *
FROM STUDENT
WHERE USN NOT IN (SELECT USN FROM BORROW);
```

**Output (sample data):**

```text
USN        NAME  ADDRESS   BRANCHID SEM
---------- ----- --------- -------- ---
1SI24MC004 Kavya Mysore    1        2  
1SI24MC005 Arun  Bangalore 1        1  
1SI24MB001 Sneha Bangalore 2        2  
```

#### Memory

JOIN branch to get the name MCA. NOT IN + subquery = rows with no match.

### Part B – Program 3

> **a. Display the USN, Student name, Branch_name, Book_name, Author_name,**  
> **Books_Borrowed_Date of 2nd sem MCA Students who borrowed books.**  
> **b. Display the number of books written by each Author.**  

#### Tables used (BRANCH, STUDENT, BOOK, AUTHOR, BORROW)

```sql
SELECT * FROM BRANCH;
SELECT * FROM STUDENT;
SELECT * FROM BOOK;
SELECT * FROM AUTHOR;
SELECT * FROM BORROW;
```

#### 3(a) 2nd sem MCA students who borrowed books

```sql
SELECT S.USN, S.Name, BR.Branchname, B.Bookname, A.Authorname, BW.Borrowed_Date
FROM STUDENT S
JOIN BRANCH BR ON S.Branchid = BR.Branchid
JOIN BORROW BW ON S.USN = BW.USN
JOIN BOOK B ON BW.Bookid = B.Bookid
JOIN AUTHOR A ON B.Authorid = A.Authorid
WHERE BR.Branchname = 'MCA' AND S.sem = 2;
```

**Output (sample data):**

```text
USN        NAME   BRANCHNAME BOOKNAME             AUTHORNAME   BORROWED_DATE
---------- ------ ---------- -------------------- ------------ -------------
1SI24MC001 Anil   MCA        DBMS Concepts        Korth        2026-08-01   
1SI24MC001 Anil   MCA        Let Us C             Kanetkar     2026-08-05   
1SI24MC001 Anil   MCA        Java Programming     Balagurusamy 2026-08-10   
1SI24MC002 Sindhu MCA        DBMS Concepts        Korth        2026-08-02   
1SI24MC002 Sindhu MCA        Operating Systems    Korth        2026-08-12   
1SI24MC003 Raj    MCA        Fundamentals of DBMS Elmasri      2026-08-03   
1SI24MC003 Raj    MCA        Let Us C             Kanetkar     2026-08-15   
```

#### 3(b) Number of books written by each author

```sql
SELECT A.Authorid, A.Authorname, COUNT(B.Bookid) AS No_of_Books
FROM AUTHOR A
LEFT JOIN BOOK B ON A.Authorid = B.Authorid
GROUP BY A.Authorid, A.Authorname;
```

**Output (sample data):**

```text
AUTHORID AUTHORNAME   NO_OF_BOOKS
-------- ------------ -----------
101      Korth        2          
102      Navathe      0          
103      Elmasri      1          
104      Kanetkar     1          
105      Balagurusamy 2          
```

#### Memory

5 tables joined through their keys. LEFT JOIN keeps authors with 0 books.

### Part B – Program 4

> **a. Display the student details who borrowed more than two books.**  
> **b. Display the student details who borrowed books of more than one Author.**  

#### Tables used (BRANCH, STUDENT, BOOK, AUTHOR, BORROW)

```sql
SELECT * FROM BRANCH;
SELECT * FROM STUDENT;
SELECT * FROM BOOK;
SELECT * FROM AUTHOR;
SELECT * FROM BORROW;
```

#### 4(a) Students who borrowed more than two books

```sql
SELECT *
FROM STUDENT
WHERE USN IN (
    SELECT USN
    FROM BORROW
    GROUP BY USN
    HAVING COUNT(*) > 2
);
```

**Output (sample data):**

```text
USN        NAME ADDRESS   BRANCHID SEM
---------- ---- --------- -------- ---
1SI24MC001 Anil Bangalore 1        2  
```

#### 4(b) Students who borrowed books of more than one author

```sql
SELECT *
FROM STUDENT
WHERE USN IN (
    SELECT BW.USN
    FROM BORROW BW
    JOIN BOOK B ON BW.Bookid = B.Bookid
    GROUP BY BW.USN
    HAVING COUNT(DISTINCT B.Authorid) > 1
);
```

**Output (sample data):**

```text
USN        NAME ADDRESS   BRANCHID SEM
---------- ---- --------- -------- ---
1SI24MC001 Anil Bangalore 1        2  
1SI24MC003 Raj  Bangalore 1        2  
```

#### Memory

GROUP BY USN + HAVING filters groups. COUNT(DISTINCT ...) counts different authors.

### Part B – Program 5

> **a. Display the Book names in descending order of their names.**  
> **b. List the details of students who borrowed the books which are all published by the same publisher.**  

#### Tables used (BRANCH, STUDENT, BOOK, AUTHOR, BORROW)

```sql
SELECT * FROM BRANCH;
SELECT * FROM STUDENT;
SELECT * FROM BOOK;
SELECT * FROM AUTHOR;
SELECT * FROM BORROW;
```

#### 5(a) Book names in descending order

```sql
SELECT Bookname
FROM BOOK
ORDER BY Bookname DESC;
```

**Output (sample data):**

```text
BOOKNAME                
------------------------
Principles of Management
Operating Systems       
Let Us C                
Java Programming        
Fundamentals of DBMS    
DBMS Concepts           
```

#### 5(b) Students whose borrowed books all have the same publisher

```sql
SELECT *
FROM STUDENT
WHERE USN IN (
    SELECT BW.USN
    FROM BORROW BW
    JOIN BOOK B ON BW.Bookid = B.Bookid
    GROUP BY BW.USN
    HAVING COUNT(DISTINCT B.Publisher) = 1
);
```

**Output (sample data):**

```text
USN        NAME   ADDRESS BRANCHID SEM
---------- ------ ------- -------- ---
1SI24MC002 Sindhu Mysore  1        2  
```

#### Memory

DESC = Z to A. One distinct publisher per student = all books from the same publisher.

### Part B – Program 6

> **Consider the following schema:**  
> **STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, total, GPA)**  
> **Perform the following:**  
> **a. Creating Tables (With and Without Constraints), Inserting/Updating/Deleting**  
> **Records in a Table, Saving (Commit) and Undoing (rollback)**  

#### Create database (Oracle uses a user/schema)

```sql
-- Run as SYSTEM (or any DBA user)
CREATE USER students IDENTIFIED BY students123;
GRANT CONNECT, RESOURCE, CREATE VIEW, UNLIMITED TABLESPACE TO students;

-- Switch to the new schema
CONNECT students/students123
```

#### Create table (with constraints)

```sql
CREATE TABLE STUDENT (
    USN NUMBER(5) PRIMARY KEY,
    name VARCHAR2(40) NOT NULL,
    date_of_birth DATE,
    branch VARCHAR2(20),
    mark1 NUMBER(3) CHECK (mark1 BETWEEN 0 AND 100),
    mark2 NUMBER(3) CHECK (mark2 BETWEEN 0 AND 100),
    mark3 NUMBER(3) CHECK (mark3 BETWEEN 0 AND 100),
    total NUMBER(4) DEFAULT 0,
    GPA NUMBER(3,2)
);
```

#### Create table without constraints

```sql
CREATE TABLE TEST2 (
    ID NUMBER,
    NAME VARCHAR2(30)
);
```

#### Insert records

```sql
INSERT INTO STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, GPA) VALUES (1001,'ANIL KUMAR',DATE '1990-01-10','MCA',85,90,88,8.90);
INSERT INTO STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, GPA) VALUES (1002,'SINDHU',DATE '1991-05-20','MCA',78,82,80,8.00);
INSERT INTO STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, GPA) VALUES (1003,'RAJ',DATE '1990-11-15','MBA',65,70,68,6.80);
INSERT INTO STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, GPA) VALUES (1004,'KAVYA',DATE '1992-03-12','BCA',90,92,95,9.30);
INSERT INTO STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, GPA) VALUES (1005,'SNEHA',DATE '1990-12-05','MBA',72,75,70,7.30);
INSERT INTO STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, GPA) VALUES (1006,'SAGAR',DATE '1991-08-10','MCA',88,85,90,8.80);

COMMIT;
```

#### Update

```sql
UPDATE STUDENT SET branch = 'MCA' WHERE USN = 1003;
UPDATE STUDENT SET branch = 'MBA' WHERE USN = 1003;
```

#### Delete

```sql
INSERT INTO STUDENT (USN, name, date_of_birth, branch, mark1, mark2, mark3, GPA)
VALUES (1099,'TEMP',DATE '1995-01-01','MCA',50,50,50,5.00);
DELETE FROM STUDENT WHERE USN = 1099;
```

#### Commit

```sql
COMMIT;
```

#### Rollback

```sql
UPDATE STUDENT SET mark1 = 0 WHERE USN = 1002;
ROLLBACK;
```

#### Table used (STUDENT)

```sql
SELECT * FROM STUDENT;
```

#### Memory

CHECK limits marks to 0-100. DEFAULT 0 gives total = 0 until we update it in Program 9.

### Part B – Program 7

> **Execute the following queries:**  
> **a. Find the GPA score of all the students.**  
> **b. Find the students who born on a particular year of birth from the date_of_birth column.**  

#### Table used (STUDENT)

```sql
SELECT * FROM STUDENT;
```

#### 7(a) GPA of all students

```sql
SELECT USN, name, GPA
FROM STUDENT;
```

**Output (sample data):**

```text
USN  NAME       GPA
---- ---------- ---
1001 ANIL KUMAR 8.9
1002 SINDHU     8  
1003 RAJ        6.8
1004 KAVYA      9.3
1005 SNEHA      7.3
1006 SAGAR      8.8
```

#### 7(b) Students born in a particular year (1990)

```sql
SELECT *
FROM STUDENT
WHERE EXTRACT(YEAR FROM date_of_birth) = 1990;
```

**Output (sample data):**

```text
USN  NAME       DATE_OF_BIRTH BRANCH MARK1 MARK2 MARK3 TOTAL GPA
---- ---------- ------------- ------ ----- ----- ----- ----- ---
1001 ANIL KUMAR 1990-01-10    MCA    85    90    88    0     8.9
1003 RAJ        1990-11-15    MBA    65    70    68    0     6.8
1005 SNEHA      1990-12-05    MBA    72    75    70    0     7.3
```

#### Memory

Oracle has no YEAR() function. Use EXTRACT(YEAR FROM date).

### Part B – Program 8

> **a. List the students who are studying in a particular branch of study.**  
> **b. Find the maximum GPA score of the student branch-wise.**  

#### Table used (STUDENT)

```sql
SELECT * FROM STUDENT;
```

#### 8(a) Students of a particular branch (MCA)

```sql
SELECT *
FROM STUDENT
WHERE branch = 'MCA';
```

**Output (sample data):**

```text
USN  NAME       DATE_OF_BIRTH BRANCH MARK1 MARK2 MARK3 TOTAL GPA
---- ---------- ------------- ------ ----- ----- ----- ----- ---
1001 ANIL KUMAR 1990-01-10    MCA    85    90    88    0     8.9
1002 SINDHU     1991-05-20    MCA    78    82    80    0     8  
1006 SAGAR      1991-08-10    MCA    88    85    90    0     8.8
```

#### 8(b) Maximum GPA branch-wise

```sql
SELECT branch, MAX(GPA) AS Max_GPA
FROM STUDENT
GROUP BY branch;
```

**Output (sample data):**

```text
BRANCH MAX_GPA
------ -------
BCA    9.3    
MBA    7.3    
MCA    8.9    
```

#### Memory

WHERE filters rows. GROUP BY + MAX() gives one answer per group.

### Part B – Program 9

> **a. Find the students whose name starts with the alphabet “S”.**  
> **b. Update the column total by adding the columns mark1, mark2, mark3.**  

#### Table used (STUDENT)

```sql
SELECT * FROM STUDENT;
```

#### 9(a) Names starting with S

```sql
SELECT *
FROM STUDENT
WHERE UPPER(name) LIKE 'S%';
```

**Output (sample data):**

```text
USN  NAME   DATE_OF_BIRTH BRANCH MARK1 MARK2 MARK3 TOTAL GPA
---- ------ ------------- ------ ----- ----- ----- ----- ---
1002 SINDHU 1991-05-20    MCA    78    82    80    0     8  
1005 SNEHA  1990-12-05    MBA    72    75    70    0     7.3
1006 SAGAR  1991-08-10    MCA    88    85    90    0     8.8
```

#### 9(b) Update total = mark1 + mark2 + mark3

```sql
UPDATE STUDENT
SET total = mark1 + mark2 + mark3;

COMMIT;

SELECT USN, name, mark1, mark2, mark3, total
FROM STUDENT;
```

**Output (sample data):**

```text
USN  NAME       MARK1 MARK2 MARK3 TOTAL
---- ---------- ----- ----- ----- -----
1001 ANIL KUMAR 85    90    88    263  
1002 SINDHU     78    82    80    240  
1003 RAJ        65    70    68    203  
1004 KAVYA      90    92    95    277  
1005 SNEHA      72    75    70    217  
1006 SAGAR      88    85    90    263  
```

#### Memory

LIKE 'S%' = starts with S. In Oracle LIKE is case-sensitive, so UPPER(name) is used to be safe.

### Part B – Program 10

> **Execute the following queries:**  
> **a. Find the students whose name ends with the alphabets “AR”.**  
> **b. Delete the student details whose USN is given as 1001.**  

#### Table used (STUDENT)

```sql
SELECT * FROM STUDENT;
```

#### 10(a) Names ending with AR

```sql
SELECT *
FROM STUDENT
WHERE UPPER(name) LIKE '%AR';
```

**Output (sample data):**

```text
USN  NAME       DATE_OF_BIRTH BRANCH MARK1 MARK2 MARK3 TOTAL GPA
---- ---------- ------------- ------ ----- ----- ----- ----- ---
1001 ANIL KUMAR 1990-01-10    MCA    85    90    88    263   8.9
1006 SAGAR      1991-08-10    MCA    88    85    90    263   8.8
```

#### 10(b) Delete student with USN 1001

```sql
DELETE FROM STUDENT
WHERE USN = 1001;

COMMIT;

SELECT * FROM STUDENT;
```

**Output (sample data):**

```text
USN  NAME   DATE_OF_BIRTH BRANCH MARK1 MARK2 MARK3 TOTAL GPA
---- ------ ------------- ------ ----- ----- ----- ----- ---
1002 SINDHU 1991-05-20    MCA    78    82    80    240   8  
1003 RAJ    1990-11-15    MBA    65    70    68    203   6.8
1004 KAVYA  1992-03-12    BCA    90    92    95    277   9.3
1005 SNEHA  1990-12-05    MBA    72    75    70    217   7.3
1006 SAGAR  1991-08-10    MCA    88    85    90    263   8.8
```

#### Memory

LIKE '%AR' = ends with AR. Always use WHERE with DELETE.
