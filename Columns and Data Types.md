## Defining Columns
A table in Oracle Database can have up to 1,000 columns. You define these when you create a table. You can also add them to existing tables.

Every column has a data type. The data type determines the values you can store in the column and the operations you can do on it. The following statement creates a table with three columns. One varchar2, one number, and one date:
```
create table this_table_has_three_columns (
  this_is_a_character_column varchar2(100),
  this_is_a_number_column    number,
  this_is_a_date_column      date
);
```
<br>

## Viewing Column Information
You can find details about the columns in your user's tables by querying user_tab_columns.

This query finds the name, type, and limits of the columns in this schema:
```
select table_name, column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns;
```
<br>

## Try It!
Complete the following statement to create the toys table with a column called toy_name:
```
create table toys (
  /* TODO */ varchar2(10)
);

select column_name, data_type, data_length
from   user_tab_columns
where  table_name = 'TOYS';
```
The query after should return the following row:
```
COLUMN_NAME   DATA_TYPE   DATA_LENGTH   
TOY_NAME      VARCHAR2         10
```
<br>

## Character Data Types
Oracle Database has three key character types:

+ varchar2
+ char
+ clob
You use these to store general purpose text.

***Varchar2***
This stores variable length text. You need to specify an upper limit for the size of these strings. In Oracle Database 11.2 and before, the maximum you can specify is 4,000 bytes. From 12.1 you can increase this length to 32,767.

***Char***
These store fixed-length strings. If the text you insert is shorter than the max length for the column, the database right pads it with spaces.

The maximum size of char is 2,000 bytes.

Only use char if you need fixed-width data. In the vast majority of cases, you should use varchar2 for short strings.

***Clob***
If you need to store text larger than the upper limit of a varchar2, use a clob. This is a character large object. It can store data up to (4 gigabytes - 1) * (database block size). In a default Oracle Database installation this is 32Tb!

The following statement creates a table with various character columns:
```
create table character_data (
  varchar_10_col   varchar2(10),
  varchar_4000_col varchar2(4000),
  char_10_col      char(10),
  clob_col         clob
);
```
```
select column_name, data_type, data_length
from   user_tab_columns
where  table_name = 'CHARACTER_DATA';
```
Each of these types also has an N variation; nchar, nvarchar2, and nclob. These store Unicode-only data. It's rare you'll use these data types.
<br>

## Numeric Data Types
The built-in numeric data types for Oracle Database are:

+ number
+ float
+ binary_float
+ binary_double
You use these to store numeric values, such as prices, weights, etc.

***Number***
This is the most common numeric data type. The format of it is:
```
number ( precision, scale )
```
The precision states the number of significant figures allowed. Scale determines the digits from the decimal point. The database rounds values that exceed the scale.

For example:
```
Min Value	Max Value
number ( 3, 2 )	-9.99	9.99
number ( 3, -2 )	-99900	99900
number ( 5 )	-99999	99999
```
If you omit the precision and scale, the number defaults to the maximum range and precision.

***Float***
This is a subtype of number. You can use it to store floating-point numbers. But we recommend that you use binary_float or binary_double instead.

***Binary_float & Binary_double***
These are floating point numbers. They can have any number of digits after the decimal point.

Binary_float is a 32-bit, single-precision floating-point number. Binary_double is a 64-bit, double-precision floating-point. The limits for these data types are:

Value	binary_float	binary_double
Maximum positive value	3.40282E+38F	1.79769313486231E+308
Minimum positive value	1.17549E-38F	2.22507485850720E-308
These also allow you to store the special values infinity and NaN (not a number).

***ANSI Numeric Types***
Oracle Database also supports ANSI numeric types, which map back to built-in types. For example:

integer => number(*, 0)
real => float(63)
The following creates a table with various numeric data types:
```
create table numeric_data (
  number_3_sf_2_dp  number(3, 2),
  number_3_sf_2     number(3, -2),
  number_5_sf_0_dp  number(5, 0),
  integer_col       integer,
  float_col         float(10),
  real_col          real,
  binary_float_col  binary_float,
  binary_double_col binary_double
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'NUMERIC_DATA';
```
Note that the columns defined with ANSI types (integer_col & real_col) are mapped to the Oracle type.
<br>

### Datetime and Interval Data Types
Oracle Database has the following datetime data types:

+ date
+ timestamp
+ timestamp with time zone
+ timestamp with local time zone
You use these to store when events happened or are planned to happen. Always use one of the above types to store datetime values. Not numeric or string types!

***Date***
Dates are granular to the second. These always include the time of day. There is no "day" data type which stores calendar dates with no time in Oracle Database.

You can specify date values with the keyword date, followed by text in the format YYYY-MM-DD. For example the following is the date 14 Feb 2018:

***date'2018-02-14'***
This is a date with a time of midnight. If you need to state the time of day too, you need to use to_date. This takes the text of your date and a format mask. For example, this returns the datetime 23 July 2018 9:00 AM:

***to_date ( '2018-07-23 09:00 AM', 'YYYY-MM-DD HH:MI AM' )***
When you store dates, the database converts them to an internal format. The client controls the display format.

***Timestamp***
If you need greater precision than dates, use timestamps. These can include up to nine digits of fractional seconds. The precision states how many fractional seconds the column stores. By default you get six digits (microseconds).

You can specify timestamp values like dates. Either use the timestamp keyword or to_timestamp with a format mask:

***timestamp '2018-02-14 09:00:00.123'
to_timestamp ( '2018-07-23 09:00:00.123 AM', 'YYYY-MM-DD HH:MI:SS.FF AM' )***
Timestamps have another advantage over dates. You can store time zone information in them. You can't store time zone details in a date.

A timestamp with time zone column stores values passed as-is. When you query a timestamp with time zone, the database returns the value you stored.

The database converts values in local time zones to its time zone. When you fetch these columns, the database returns it in the time zone of the session.

***Time Intervals***
You can store time durations with intervals. Oracle Database has two interval types: year to month and day to second.

You can add or subtract intervals from dates, timestamps or equivalent intervals. But the intervals are incompatible! You can't combine a day to second interval with a year to month one. This is because the number of days varies between months and years.

The following creates a table with the various datetime data types:
```
create table datetime_data (
  date_col                      date,
  timestamp_with_3_frac_sec_col timestamp(3),
  timestamp_with_tz             timestamp with time zone,
  timestamp_with_local_tz       timestamp with local time zone,
  year_to_month_col             interval year to month,
  day_to_second_col             interval day to second
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'DATETIME_DATA';
```
<br>

### Binary Data Types
You use binary data to store in its original format. These are usually other files, such as graphics, sound, video or Word documents. There are two key binary types: raw and blob.

***Raw***
Like with character data, raw is for smaller items. You specify the maximum length of data for each column. It has a maximum limit of 2,000 bytes up to 11.2 and 32,767 from 12.1.

***Blob***
Blob stands for binary large object. As with clob, the maximum size you can store is (4 gigabytes - 1) * (database block size).

The following creates a table with binary data type columns:
```
create table binary_data (
  raw_col  raw(1000),
  blob_col blob
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'BINARY_DATA';
```
<br>

## Try It!
Complete the following statement to create a table with the following columns:
```
brick_id of type number(20, 0)
colour of type varchar2, max size 10
price of type number with precision 10 and scale 2
purchased_date of type date
create table bricks (
  /* TODO */
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'BRICKS';
```
When successful, the query should return the following rows:
```
COLUMN_NAME      DATA_TYPE   DATA_LENGTH   DATA_PRECISION   DATA_SCALE   
BRICK_ID         NUMBER                 22               20            0 
COLOUR           VARCHAR2               10           <null>       <null>
PRICE            NUMBER                 22               10            2 
PURCHASED_DATE   DATE                    7           <null>       <null>
```
<br>

## Adding Columns to Existing Tables
You add columns to an existing table with alter table. You can add as many as you want (up to the 1,000 column table limit):

The following adds two columns to the table this_table_has_three_columns. One timestamp and one blob:
```
alter table this_table_has_three_columns add (
  this_is_a_timestamp_column    timestamp, 
  this_is_a_binary_large_object blob
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'THIS_TABLE_HAS_THREE_COLUMNS';
```
<br>

## Removing Columns from a Table
You can also remove columns from a table. To get rid of a column from a table, alter the table again, this time with the drop clause.

The following removes the columns you added to this_table_has_three_columns in the previous step:
```
alter table this_table_has_three_columns drop (
  this_is_a_timestamp_column, 
  this_is_a_binary_large_object
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'THIS_TABLE_HAS_THREE_COLUMNS';
```
Note this is a one-way operation! After you drop a column there is no "undrop" command. If you want to get the column back, you have to restore it from a backup.

Dropping columns is an expensive operation. It can take a long time to complete on large tables. So always triple, quadruple check before running this!

<br>

## Try It!
Complete the alter table statements to:

Add the weight as a number with precision 8 and scale 1
Remove cuddliness factor
```
drop table toys;
create table toys (
  toy_id            integer,
  toy_name          varchar2(100),
  cuddliness_factor integer
);

alter table toys add /* TODO */;

alter table toys drop ( /* TODO */ );

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'TOYS';
```
The query at the end should return the following rows:
```
COLUMN_NAME   DATA_TYPE   DATA_LENGTH   DATA_PRECISION   DATA_SCALE   
TOY_ID        NUMBER                 22           <null>            0 
TOY_NAME      VARCHAR2              100           <null>       <null> 
WEIGHT        NUMBER                 22                8            1 
```
