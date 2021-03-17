<h1 align='center'>Querying and Filtering Rows</h1>

**Prerequisite SQL**
```
create table toys (
  toy_name varchar2(100),
  colour   varchar2(10),
  price    number(10, 2)
);

insert into toys values ( 'Sir Stripypants', 'red', 0.01 );
insert into toys values ( 'Miss Smelly_bottom', 'blue', 6.00 );
insert into toys values ( 'Cuteasaurus', 'blue', 17.22 );
insert into toys values ( 'Mr Bunnykins', 'red', 14.22 );
insert into toys values ( 'Baby Turtle', 'green', null );

commit;
```
You access rows in a database table with the select statement. This returns data to the client. It has two core parts: select and from.

In the from clause you list the tables you want to get the rows from. And in select you state which columns you want to see the values of.

Placing asterisk (star) returns all the visible columns in the table. So to get all the rows and visible columns in the toys table, use:
```
select * from toys;
```
This excludes any invisible columns in the table. These are often system-generated columns. From Oracle Database 12c you can set the visibility of a column.

To view invisible columns you have to list them in the select clause. Here you can also restrict your query to fewer columns or change the order they appear. Do this by naming the columns you want like so:
```
select toy_name, price from toys;
```
It is good practice to only select the columns you need. This reduces the volume of data sent over the network. Which can make your application faster.

It also makes your code more resilient. If you use *, your code may error if someone adds or removes columns from the table.
<br>
## Filtering Data
It's rare you want to return all the rows from a table. Usually you only want those matching some search criteria.

You do this filtering in the where clause. The query only returns rows where the whole clause is true. For example, this gets the row that stores "Sir Stripypants" in the column toy_name:
```
select * from toys
where  toy_name = 'Sir Stripypants';
```
A condition can match many rows. The database will return all of them. The rows for Sir Stripypants & Mr Bunnykins both have the colour red. So this query fetches them both:
```
select * from toys
where  colour = 'red';
```
<br>
## Combining Criteria
You can combine many filters with AND & OR.

**AND**
This returns the rows where both conditions are true.

The following searches for rows where the toy_name is Sir Stripypants. And the colour is green. Sir Stripypants is red, so this query returns nothing:
```
select * from toys
where  toy_name = 'Sir Stripypants'
and    colour = 'green';
```
**OR**
OR fetches all the rows where either of the criteria are true. Baby Turtle has the colour green. So this query gets the row for them and Sir Stripypants:
```
select * from toys
where  toy_name = 'Sir Stripypants' or
       colour = 'green';
```
**Order of Precedence**
AND has higher priority than OR. So if you include both in a where clause, the order you place them affects the results.

For example, the following two queries search for the same values from each column. But they return different rows:
```
select * from toys
where  toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turtle'
and    colour = 'green';

select * from toys
where  colour = 'green'
and    toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turtle';
```
Why? Well the database processes the filters in different orders.

The first query searches for rows where:

- The colour is green and the toy_name is "Baby Turtle"
- Or the toy_name is "Mr Bunnykins"
But the second looks for rows where:

- The colour is green and the toy_name is "Mr Bunnykins"
- Or the toy_name is "Baby Turtle"
So the first will always return Mr Bunnykins. The second will always return Baby Turtle. Mr Bunnykins is red. So you get this row in the first query, but not the second.

To avoid confusion in queries combining AND with OR, use parentheses. The database processes conditions inside the brackets first. So the following two queries both search for rows where:

- The toy_name is "Mr Bunnykins" or "Baby Turtle"
- And the colour is green
```
select * from toys
where  ( toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turtle' )
and    colour = 'green';

select * from toys
where  colour = 'green'
and    ( toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turtle' );
```



