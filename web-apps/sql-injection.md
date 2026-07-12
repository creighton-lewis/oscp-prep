
# Connecting

```
mysql -u root -p

```

```
mysql -u user -ppass
```

```
mysql -u user -h remote.host.htb -P 3306 -p
```

```
mysql -u user -h remote.host.htb -P 3306 -p --ssl=DISABLE
```

# Enumeration

```
nuclei -u http://url -tag mysql
```

```
sudo nmap -sV -sC 3306
```
# External
Table Manipulation
```
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value); 
INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');
ALTER TABLE logins ADD newcolumn INT: 
ALTER TABLE logins MODIFY newerColumn DATE;
```
```
UPDATE logins SET password = 'change_password' WHERE id > 1;
Internal
Database Version
SELECT @@version
SELECT POW(1,1)
SELECT SLEEP(5) 
Table Selection
SELECT * FROM table_name; *

\
```
```
SELECT * FROM logins ORDER BY password;
SELECT * FROM logins ORDER BY password DESC, id ASC;
```

**Limiting Results**

SELECT * FROM logins LIMIT 2;
SELECT * FROM logins LIMIT 1, 2;
Where Results

SELECT * FROM table_name WHERE <condition>;
SELECT * FROM logins WHERE id > 1;
SELECT * FROM logins where username = 'admin'
Like

SELECT * FROM logins WHERE username LIKE 'admin%';
SELECT * FROM logins WHERE username like '___';
SELECT * FROM logins WHERE username != 'john' AND id > 1;

**Column Selection**
```
SELECT * from information_schema.columns
```


# Exploitation
## Logic Suberversion

#### Comments
Comments can be added to prevent certain logic from being executed. Anything found after a -- or ## is not executed
Parentheses can also be used to ensure certain conditions are checked before others

**Example**
```
 SELECT username FROM logins; --
 SELECT username FROM logins; -- - 
 SELECT username FROM logins; #
```

#### Union Clause / Injection 
Union clauses combine results from SELECT statements
Data types from each column MUST be the same, and the number of columns in each table MUST also match

1. Detect number of columns

- Use the function ORDER BY until there is an error that says a column does not exist

2. Determine injection location

#### Data Extraction
```
```
