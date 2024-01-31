# Little-Lemon-Restaurant-Database
This project is the Database Engineer Capstone guided by Meta. It passes through many steps:
1. [Setting a database](#1-setting-a-database)
   * [The information required to be stored in this database](#the-information-required-to-be-stored-in-this-database)
   * [Main entites and related attributes](#main-entites-and-related-attributes)
   * [Normalization Process](#normalization-process)
   * [Final Desgin of the Database](#final-desgin-of-the-database)
   * [Implement the databse into MySQL Sever](#implement-the-databse-into-mysql-sever)
2. [Creating reports](#2-creating-reports)
	* [The orders placed in the restaurant](#task-1--the-orders-placed-in-the-restaurant)
	* [The customers with specific orders](#task-2--all-customers-with-orders-that-cost-more-than-150)
	* [The best selling menu items](#task-3--all-menu-items-for-which-more-than-2-orders-have-been-placed)
	* [Query Optimization](#query-optimization)
	* [Displays the maximum ordered quantity in the Orders table](#task-1--displays-the-maximum-ordered-quantity-in-the-orders-table)
	* [Return information about an order](#task-2--return-information-about-an-order)
	* [Delete a specific order with its id](#task-3--delete-a-specific-order-with-its-id)
	* [Check the booking status of any table in the restaurant](#task-4--check-the-booking-status-of-any-table-in-the-restaurant)
	* [Verify a booking, and decline any reservations for tables that are already booked under another name](#task-5--verify-a-booking-and-decline-any-reservations-for-tables-that-are-already-booked-under-another-name)
	* [Update existing bookings in the booking table](#task-6--update-existing-bookings-in-the-booking-table)
	* [Cancel a booking](#task-7--cancel-a-booking)
3. [Creating database clients](#3-creating-database-clients)
4. [Creating Tableau Dashboard](#4.-create-a-tableau-dashboard#)
<hr>

# 1. Setting a database #

## The information required to be stored in this database ##

1. Bookings: To store information about booked tables in the restaurant including booking id, date and table number.

2. Orders: To store information about each order such as order date, quantity and total cost.

3. Order delivery status: To store information about the delivery status of each order such as delivery date and status.

4. Menu: To store information about cuisines, starters, courses, drinks and desserts.

5. Customer details: To store information about the customer names and contact details.

6. Staff information: Including role and salary.

<hr>

## Main entites and related attributes ##

* Customers:
   * PK &rarr; Customer ID
   * Attributes &rarr; Full Name - Phone Number

* Staff:
   * PK &rarr; Staff ID
   * Attributes &rarr; Name - Role - Salary - Address - Contact Number - Email

* Mene Items:
  * PK &rarr; Item ID
  * Attributes &rarr; Item name - Category - Cuisine - Price

* Bookings:
  * PK &rarr; Booking ID
  * Attributes &rarr; Booking date - Customer - Table number - Number of Persons - Staff

* Orders:
  * PK &rarr; Order ID
  * Attributes &rarr; Table number - Order date - Total cost - Booking ID - Items and Quantity

* Orders Delivery Status:
  * PK &rarr; Order ID
  * Attributes &rarr; Order ID - Delivery Date - Delivery Status

<hr>

## Normalization Process ##

* The tables Orders have multi-valued attrbiutes such as Items and Quantities
* The solution is to split this table into 2 tables:
    1. Orders &rarr; The original table without the items and quantity
    2. Orders_Details
        * PK &rarr; Order ID & Item ID
        * Attributes &rarr; Quantity

<hr>

## Final Desgin of the Database ##

This design was implemented using MySQL Workbench tool.

<p align="center">
<img src="https://user-images.githubusercontent.com/70551007/236650190-ff8b1a48-4d9b-446d-85c7-b7b9cc7a8be6.png">
</p>

<hr>

## Implement the database into MySQL Sever ##

Using the _Forward Engineer_ feature in MySQL Workbench, we can implement the database into the server using its ER-Diagram.

Here is some snippets of the code generated by the _Forward Engineer_ tool.

* Creating the whole schema
    ```sql
    CREATE SCHEMA IF NOT EXISTS `little_lemon_db` DEFAULT CHARACTER SET utf8 ;
    USE `little_lemon_db` ;
    ```

* Creating the __Bookings__ table
    ```sql
    CREATE TABLE IF NOT EXISTS `little_lemon_db`.`Bookings` (
    `Booking_ID` INT NOT NULL,
    `Booking_Date` DATETIME NOT NULL,
    `Customer_ID` INT NOT NULL,
    `Table_number` INT NOT NULL,
    `Number_of_Persons` INT NOT NULL,
    `Staff_ID` INT NOT NULL,
    PRIMARY KEY (`Booking_ID`),
    INDEX `FK_customers_in_bookings_idx` (`Customer_ID` ASC) VISIBLE,
    INDEX `FK_staff_in_bookings_idx` (`Staff_ID` ASC) VISIBLE,
    CONSTRAINT `FK_customers_in_bookings`
        FOREIGN KEY (`Customer_ID`)
        REFERENCES `little_lemon_db`.`Customers` (`Customer_ID`)
        ON DELETE NO ACTION
        ON UPDATE NO ACTION,
    CONSTRAINT `FK_staff_in_bookings`
        FOREIGN KEY (`Staff_ID`)
        REFERENCES `little_lemon_db`.`Staff` (`Staff_ID`)
        ON DELETE NO ACTION
        ON UPDATE NO ACTION)
    ENGINE = InnoDB;
    ```

* Creating the __Menu__ table
    ```sql
    CREATE TABLE IF NOT EXISTS `little_lemon_db`.`Menu` (
    `Item_ID` INT NOT NULL,
    `Name` VARCHAR(45) NOT NULL,
    `Category` VARCHAR(45) NOT NULL,
    `Cuisine` VARCHAR(45) NOT NULL,
    `Price` INT NOT NULL,
    PRIMARY KEY (`Item_ID`))
    ENGINE = InnoDB;
    ```

> The whole code file is at the _database-codes_ folder in the repo.

<hr>

# 2. Creating reports #

Little Lemon Restaurant needs some reports to monitor:
* The orders placed in the restaurant.
* The customers with specific orders.
* The menu items with specific number of orders.

And some features such as:
* Delete a specific order
* Check bookings
* Add valid bookings
* Update bookings
* Cancel Bookings

And to do those tasks we need to use some clauses and features from SQL like _Views_, _Stored Procedures_, _Triggers_ and _Prepare Statements_.

<hr>

## Task 1 : The orders placed in the restaurant ##

Little Lemon needs to know some information about the order placed in the restaurant like OrderID, Quantity, and Total Cost. It results in:

```sql
mysql> SELECT * FROM OrdersView;
+---------+----------+------------+
| OrderID | Quantity | Total_Cost |
+---------+----------+------------+
|       1 |        7 |        890 |
|       2 |        6 |        800 |
+---------+----------+------------+
2 rows in set (0.00 sec)
```

<hr>

## Task 2 : All customers with orders that cost more than $150 ##

Little Lemon needs to know some information about the customers with specific orders. It results in:

```sql
SELECT C.Customer_ID, C.Full_Name, O.Order_ID, (M.Item_ID * M.Price) AS Cost, M.Name AS MenuName
FROM Customers AS C
INNER JOIN Bookings AS B
	ON B.Customer_ID = C.Customer_ID
INNER JOIN Orders AS O
	ON O.Booking_ID = B.Booking_ID
INNER JOIN Orders_Details AS OD
	ON OD.OrderID = O.Order_ID
INNER JOIN Menu AS M
	ON M.Item_ID = OD.Item_ID
WHERE O.Total_Cost > 150;

+-------------+---------------+----------+------+----------+
| Customer_ID | Full_Name     | Order_ID | Cost | MenuName |
+-------------+---------------+----------+------+----------+
|           1 | Rabia Mendoza |        1 |  150 | Pasta    |
|           1 | Rabia Mendoza |        1 |  140 | Salad    |
|           2 | Aayan Chaney  |        2 |  150 | Pasta    |
|           2 | Aayan Chaney  |        2 |  300 | Fried    |
+-------------+---------------+----------+------+----------+
4 rows in set (0.06 sec)
```

<hr>

## Task 3 : All menu items for which more than 2 orders have been placed ##

Little Lemon needs to know some information about the menu items for which more than 2 orders have been placed. It results in:

```sql
SELECT Name AS MenuName FROM Menu
WHERE Name = ANY(SELECT M.Name FROM Menu AS M
					INNER JOIN Orders_Details AS OD
						ON OD.Item_ID = M.Item_ID);

+----------+
| MenuName |
+----------+
| Pasta    |
| Salad    |
| Fried    |
+----------+
3 rows in set (0.14 sec)
```

<hr>

## Query Optimization ##

When working with MySQL databases, the response time, or turnaround time, is extremely important. It’s particularly important in terms of how long the database takes to respond to your SQL queries. As data volumes grow, and the data requirements grow increasingly more complex, then performance becomes more important for a better end-user experience.

Database optimization is the best way to reduce database system response time. Response time is the time taken to transmit a query, process it, and transmit the response or information back to the user.  

We can optimize database queries using _stored procedures_ and _prepared statements_.

<hr>

## Task 1 : Displays the maximum ordered quantity in the Orders table ##

This was done using a _Stored Procedure_ called __GetMaxQuantity()__. It results in:

```sql
mysql> CALL GetMaxQuantity();
+-----------------------+
| Max Quantity in Order |
+-----------------------+
|                     7 |
+-----------------------+
1 row in set (0.08 sec)

Query OK, 0 rows affected (0.08 sec)
```

<hr>

## Task 2 : Return information about an order #

This was done using a _Prepared Statement_ called __GetOrderDetail__. It results in:

```sql
mysql> SET @id = 1;
Query OK, 0 rows affected (0.05 sec)

mysql> EXECUTE GetOrderDetail USING @id;
+---------+----------+------------+
| OrderID | Quantity | Total_Cost |
+---------+----------+------------+
|       1 |        7 |        890 |
+---------+----------+------------+
1 row in set (0.04 sec)
```

<hr>

## Task 3 : Delete a specific order with its id ##

This was done using a _Stored Procedure_ called __CancelOrder__. It takes the Order_ID and delete the order from the __Orders__, the __Orders_Details__, and the __Orders_Delivery_Status__ tables.

```sql
mysql> CALL CancelOrder(1);
+---------------------+
| Confirmation        |
+---------------------+
| Order 1 is canceled |
+---------------------+
1 row in set (0.32 sec)

Query OK, 0 rows affected (0.32 sec)
```

<hr>

## Task 4 : Check the booking status of any table in the restaurant ##

This was done using a _Stored Procedure_ called __CheckBooking__. It takes 2 input:
1. Booking date
2. Table number
And then check if the booking already exists in the __Bookings__ table or not.

```sql
mysql> CALL CheckBooking("2022-08-10 12:00:00", 5);
+----------------------------+
| Booking Status             |
+----------------------------+
| Table 5 is already booked. |
+----------------------------+
1 row in set (0.05 sec)

Query OK, 0 rows affected (0.05 sec)
```

<hr>

## Task 5 : Verify a booking, and decline any reservations for tables that are already booked under another name ##

This was done using a _Stored Procedure_ called __AddValidBooking__. It takes 4 inputs:
1. Booking date
2. Table number
3. Customer ID
4. Number of persons

And check if there's an existing booking at the same time on the same table number, it prints that there's already an existing booking. Else, it stores this booking into the __Bookings__ table.

* If there's already a booking at this time on the same table number.

	```sql
	mysql> CALL AddValidBooking("2022-08-10 12:00:00", 5, 1, 5);
	+-----------------------------------------------+
	| Booking Status                                |
	+-----------------------------------------------+
	| Table 5 is already booked. Booking cancelled. |
	+-----------------------------------------------+
	1 row in set (0.03 sec)

	Query OK, 0 rows affected (0.03 sec)
	```

* If the booking date is different from all the bookings.

	```sql
	mysql> CALL AddValidBooking("2022-10-10 12:00:00", 5, 1, 5);  
	+----------------------------+
	| Booking Status             |
	+----------------------------+
	| Table 5 is booked for you. |
	+----------------------------+
	1 row in set (0.07 sec)

	Query OK, 0 rows affected (0.07 sec)

	mysql> SELECT * FROM Bookings;
	+------------+---------------------+-------------+--------------+-------------------+----------+
	| Booking_ID | Booking_Date        | Customer_ID | Table_number | Number_of_Persons | Staff_ID |
	+------------+---------------------+-------------+--------------+-------------------+----------+
	|          1 | 2022-08-10 12:00:00 |           1 |            5 |                10 |        2 |
	|          2 | 2022-09-15 03:30:00 |           2 |            4 |                 4 |        2 |
	|          3 | 2022-10-10 12:00:00 |           1 |            5 |                 5 |        2 |
	+------------+---------------------+-------------+--------------+-------------------+----------+
	3 rows in set (0.00 sec)
	```

* If the table number is different from all the bookings.

	```sql
	mysql> CALL AddValidBooking("2022-10-10 12:00:00", 2, 1, 5); 
	+----------------------------+
	| Booking Status             |
	+----------------------------+
	| Table 2 is booked for you. |
	+----------------------------+
	1 row in set (0.03 sec)

	Query OK, 0 rows affected (0.04 sec)

	mysql> SELECT * FROM Bookings;
	+------------+---------------------+-------------+--------------+-------------------+----------+
	| Booking_ID | Booking_Date        | Customer_ID | Table_number | Number_of_Persons | Staff_ID |
	+------------+---------------------+-------------+--------------+-------------------+----------+
	|          1 | 2022-08-10 12:00:00 |           1 |            5 |                10 |        2 |
	|          2 | 2022-09-15 03:30:00 |           2 |            4 |                 4 |        2 |
	|          3 | 2022-10-10 12:00:00 |           1 |            5 |                 5 |        2 |
	|          4 | 2022-10-10 12:00:00 |           1 |            2 |                 5 |        2 |
	+------------+---------------------+-------------+--------------+-------------------+----------+
	4 rows in set (0.05 sec)
	```

<hr>

## Task 6 : Update existing bookings in the booking table ##

This was done using a _Stored Procedure_ called __UpdateBooking__. It takes 2 inputs:
1. Booking ID
2. The new booking date

* If the booking already exists

	```sql
	mysql> CALL UpdateBooking(2, "2022-09-20 3:30");
	+-----------------------+
	| Confirmation          |
	+-----------------------+
	| Booking 2 is updated. |
	+-----------------------+
	1 row in set (0.09 sec)

	Query OK, 0 rows affected (0.10 sec)

	mysql> SELECT * FROM Bookings;
	+------------+---------------------+-------------+--------------+-------------------+----------+
	| Booking_ID | Booking_Date        | Customer_ID | Table_number | Number_of_Persons | Staff_ID |
	+------------+---------------------+-------------+--------------+-------------------+----------+
	|          1 | 2022-08-10 12:00:00 |           1 |            5 |                10 |        2 |
	|          2 | 2022-09-20 03:30:00 |           2 |            4 |                 4 |        2 |
	|          3 | 2022-10-10 12:00:00 |           1 |            5 |                 5 |        2 |
	|          4 | 2022-10-10 12:00:00 |           1 |            2 |                 5 |        2 |
	+------------+---------------------+-------------+--------------+-------------------+----------+
	4 rows in set (0.00 sec)
	```

* If the booking doesn't exist.

	```sql
	mysql> CALL UpdateBooking(7, "2022-09-20 3:30"); 
	+---------------------------------------+
	| Confirmation                          |
	+---------------------------------------+
	| There is no booking with booking id 7 |
	+---------------------------------------+
	1 row in set (0.03 sec)

	Query OK, 0 rows affected (0.03 sec)
	```

<hr>

## Task 7 : Cancel a booking ##

This was done using a _Stored Procedure_ called __CancelBooking__. It takes the Booking ID which the customer wants to cancel, and delete the booking from the __Bookings__ table, as well as delete the order related to this booking from the __Orders__, __Orders_Details__, and __Orders_Delivery_Status__.

* If the booking already exists.

	```sql
	mysql> CALL CancelBooking(2);
	+-------------------------+
	| Confirmation            |
	+-------------------------+
	| Booking 2 is cancelled. |
	+-------------------------+
	1 row in set (0.29 sec)

	Query OK, 0 rows affected (0.29 sec)
	```

* If the booking doesn't exists.

	```sql
	mysql> CALL CancelBooking(8); 
	+---------------------------------------+
	| Confirmation                          |
	+---------------------------------------+
	| There is no booking with booking id 8 |
	+---------------------------------------+
	1 row in set (0.00 sec)

	Query OK, 0 rows affected (0.01 sec)
	```

<hr>

# 3. Creating database clients #

Little Lemon Restaurant needs a database client to deal with the database using a Python Appliction. To do this task we need to connect python to the database, and this can be done using ```mysql-connector-python```.

First, I've imported the library then established the connection.

```python
connection = connector.connect(
    user = ENV_DATABASE_USER,
    password = ENV_DATABASE_PASSWORD,
    host = ENV_DATABASE_HOST,
    port = ENV_DATABASE_PORT)
```

After that, I've created a cursor to link the Pyhon Application to the database, and use the database.

```python
cursor = connection.cursor()

use_database = """USE Little_Lemon_DB;"""
cursor.execute(use_database)
```

Finally, Little Lemon now has a database client, and can monitor the information they need through a Python Application.

<hr>

I've done some tasks using this application:

1. Show the tables of the database

    ```python
    show_tables_query = """SHOW TABLES;""" 
    cursor.execute(show_tables_query)
    results = cursor.fetchall()

    for i, result in enumerate(results):
        print("Table no. ", i+1, ": ", result[0])
    ```

    and this gave me the following results:

    ```
    Table no.  1 :  bookings
    Table no.  2 :  customers
    Table no.  3 :  menu
    Table no.  4 :  orders
    Table no.  5 :  orders_delivery_status
    Table no.  6 :  orders_details
    Table no.  7 :  ordersview
    Table no.  8 :  staff
    ```

2. Extract some information about the customers who paid more than 60$ for the purpose of promotional campaign

    ```python
    promotional_campaign = """
        SELECT C.Full_Name, C.Phone_Number, O.Total_Cost
        FROM Customers AS C
        INNER JOIN Bookings AS B
            ON B.Customer_ID = C.Customer_ID
        INNER JOIN Orders AS O
            ON O.Booking_ID = B.Booking_ID
        WHERE O.Total_Cost > 60;
    """

    cursor.execute(promotional_campaign)

    results = cursor.fetchall()

    print("Information about customers for the purpose of promotional campaign:")

    for i, result in enumerate(results):
        print("Customer no. ", i+1, ": ", result[0], "whose contact is (", result[1], ") has paid ", result[2], "$.")
    ```

    and this gave me the following results:

    ```
    Information about customers who win promotional_campaign:
    Customer no.  1 :  Rabia Mendoza whose contact is ( 0123456789 ) has paid  890 $.
    Customer no.  2 :  Aayan Chaney whose contact is ( 0129876543 ) has paid  800 $.
    ```
# 4. Create a Tableau Dashboard #
In this task, we aimed to create interactive dashboards to display sales and profits for Little Lemon based on the provided guidance.

## Customer Sales Bar Chart ##

1. **Create a Bar Chart:**
   - Drag the relevant fields (`Customer Name` and `Sales`) into the appropriate shelves.
   - Set up a bar chart to display customer sales.

2. **Color Scheme:**
   - Apply a suitable color scheme to enhance readability.

3. **Filter Sales Data:**
   - Apply a filter to include sales with at least $70.

4. **Chart Naming:**
   - Name the chart "Customer Sales."

5. **Tooltip Configuration:**
   - Configure the tooltip to display customer names and sale figures upon hovering over a bar.
![Customer Sales](./Tableau%20Dashboard/Customer%20Sales.png)

## Sales Trend Line Chart ##

1. **Create a Line Chart:**
   - Drag the relevant fields (`Date` and `Sales`) into the appropriate shelves.
   - Set up a line chart to visualize the sales trend.

2. **Color Scheme:**
   - Apply a suitable color scheme for better visualization.

3. **Exclude 2023:**
   - Filter the data to exclude the year 2023.

4. **Chart Naming:**
   - Name the chart "Profit Chart."
![Profit Chart](./Tableau%20Dashboard/Profit%20Chart.png)

## Sales Bubble Chart ##

1. **Create a Bubble Chart:**
   - Drag the relevant fields (`Customer Name`, `Sales`, and `Profit`) into the appropriate shelves.
   - Set up a bubble chart to visualize sales for all customers.

2. **Color Scheme:**
   - Apply a suitable color scheme.

3. **Chart Naming:**
   - Name the chart "Sales Bubble Chart."

4. **Tooltip Configuration:**
   - Configure the tooltip to display customer name, profit, and sales upon hovering over a bubble.
![Sales Bubble Chart](./Tableau%20Dashboard/Sales%20Bubble%20Chart.png)

## Cuisine Sales and Profits Bar Chart ##

1. **Create a Bar Chart:**
   - Drag the relevant fields (`Cuisine`, `Sales`, and `Profit`) into the appropriate shelves.
   - Set up a bar chart to compare sales of Turkish, Italian, and Greek cuisines.

2. **Color Scheme:**
   - Apply a suitable color scheme.

3. **Filter Data:**
   - Display sales data for 2020, 2021, and 2022 only.

4. **Sort Data:**
   - Sort the bars in descending order by the sum of sales.

5. **Chart Naming:**
   - Name the chart "Cuisine Sales and Profits."

![Cuisine Sales and Profits](RendyAdiyana/Little_Lemon_DB_Capstone_Project/Tableau%20Dashboard/Cuisine%20Sales%20and%20Profits.png)

## Interactive Dashboard ##

1. **Combine Bar Chart and Bubble Chart:**
   - Create an interactive dashboard that includes the "Customer Sales" bar chart and the "Sales Bubble Chart."

2. **Interactivity:**
   - Configure the dashboard to display customer name, sales, and profit figures in the bubble chart when clicking a bar and rolling over the related bubble.

3. **Chart Naming:**
   - Name the interactive dashboard.
![Dashboard](./Tableau%20Dashboard/Dashboard.png)

By completing these tasks, Little Lemon now has interactive dashboards that provide insights into their sales, profits, and customer-related information. These visualizations empower them to make informed decisions and enhance their business performance.
