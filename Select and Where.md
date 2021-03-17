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
## Selecting Rows
<br>
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
## Combining Criteria 
<br>
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
## Lists of Values
Often you want to get rows where a column matches any value in a list. You can do this by ORing these conditions together. For example, the following finds rows where the colour is red or green:
```
select * from toys
where  colour = 'red' or
       colour = 'green';
```
But this is a pain to write if you have a large number of values!

Luckily you can simplify this with IN. Place the list of values in parentheses. Then check if the column is IN this list. So this query is the same as the one above, finding all the rows where the colour is red or green:
```
select * from toys
where  colour in ( 'red' , 'green' );
```
## Ranges of Values

You can also find all the rows matching a range of values with inequalities such as <, >=, etc.

For example, to find all the toys that cost less than 10, use:
```
select * from toys
where  price < 10;
```
Or those with a price greater than or equal to 6 with:
```
select * from toys
where  price >= 6;
```
You can also use the condition between. This returns rows with values from a lower to an upper bound. This is inclusive, so it returns rows with values matching either limit. So the following gets all the data with a price equal to 6, 20, or any value between these two:
```
select * from toys
where  price between 6 and 20;
```
It is the same as the following query:
```
select * from toys
where  price >= 6
and    price <= 20;
```
If you want to exclude rows at either boundary, you need to write the separate tests. For example, to get all the rows where the price is greater than 6 and less than or equal to 20, use:
```
select * from toys
where  price > 6 
and    price <= 20;
```
## Wildcards

When searching strings, you can find rows matching a pattern using LIKE. This has two wildcard characters:

+ Underscore (_) matches exactly one character
+ Percent (%) matching zero or more characters
You can place these either side of the characters you're searching for. So this finds all the rows that have a colour starting with b:
```
select * from toys
where  colour like 'b%';
```
And this all the rows with colours ending in n:
```
select * from toys
where  colour like '%n';
```
Underscore matches exactly one character. So the following finds all the rows with toy_names eleven characters long:
```
select * from toys
where  toy_name like '___________';
```
Percent is true even if it matches no characters. So the following tests to search for colours containing the letter "e" all return different results:
```
select * from toys
where  colour like '_e_';

select * from toys
where  colour like '%e%';

select * from toys
where  colour like '%_e_%';
```
This is because these searches work as follows:

+ _e_ => any colour with exactly one character either side of e (red)
+ %e% => any colour that contains e anywhere in the string (red, blue, green)
+ %_e_% => any colour with at least one character either side of e (red, green)

**Searching for Wildcard Characters**

You may want to find rows that contain either underscore or percent. For example, if you want to see all the rows that include underscore in the toy_name, you might try:
```
select * from toys
where  toy_name like '%_%';
```
But this returns all the rows!

This is because it sees underscore as the wildcard. So it looks for all rows that have at least one character in toy_name.

To avoid this, you can use escape characters. Place this character before the wildcard. Then state what it is in the escape clause after the condition. This can be any character. But usually you'll use symbols you're unlikely to search for. Such as backslash \ or hash #.

So both the following find Miss Smelly_bottom, the only toy_name that includes an underscore:
```
select * from toys
where  toy_name like '%\_%' escape '\';

select * from toys
where  toy_name like '%#_%' escape '#';
```
## Null

The price for Baby Turtle is null. This is neither equal to nor not equal to anything! The result of comparing a value to null is unknown.

Where clauses only return rows where the tests are true. So if you search for rows where the price equals null, you get no data:
```
select * from toys
where  price = null;
```
To find rows storing null values, you must use the "is null" condition:
```
select * from toys
where  price is null;
```
## Negation

You can return the opposite of most conditions by placing NOT before it. For example, to find all the toys that aren't green, you can do:
```
select *
from   toys 
where  not colour = 'green';
```
You can get the same result by changing equals to either of the not equal conditions, != or <>:
```
select *
from  toys 
where colour <> 'green';
```
One exception to this is null. Searching for rows that are NOT equal to null still returns nothing:
```
select *
from   toys 
where  not colour = null;

select *
from   toys 
where  colour <> null;
```
To get all the rows with a non-null value, you must use the is not null condition:
```
select *
from   toys 
where  colour is not null;
```
## Try It!

Complete the following query to find the rows where:

The colour is not green
The price is not equal to 6
```
select toy_name
from   toys
where  /* TODO */
```
This should return the following rows:

**TOY_NAME**        
Sir Stripypants   
Cuteasaurus       
Mr Bunnykins     
How many ways can you think of to write this query?



