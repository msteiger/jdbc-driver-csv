# JDBC driver for CSV

[Follow me on twitter](https://twitter.com/xbib)

This driver is derived from the CsvJdbc project http://csvjdbc.sourceforge.net/

JDBC driver for CSV is a read-only JDBC driver that uses Comma Separated Value (CSV) files
or DBF files as database tables. It is ideal for writing data import programs
or analyzing log files.

The driver enables a directory or a ZIP file containing CSV or DBF files
to be accessed as though it were a database containing tables.
However, as there is no real database management system behind the scenes,
not all JDBC functionality is available.

# Maven project usage

    <repositories>
        <repository>
            <id>xbib</id>
            <url>http://xbib.org/repository</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.xbib.jdbc</groupId>
            <artifactId>jdbc-driver-csv</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>

# Documentation

This driver accepts all types of CSV files defined by RFC 4180.

The URL syntax for the driver URL is as follows

    jdbc:xbib:csv:<foldername>[?<property=value]",
    
File names are mapped to table names. By default, the extension `.csv` is assumed.
Example:

    "jdbc:xbib:csv:myDataFolder?quotechar=~",

This will provide the content of the folder *myDataFolder* through the JDBC interface.

## Queries

This driver accepts only SQL SELECT queries from a single table and does not
support INSERT, UPDATE, DELETE or CREATE statements. Joins between tables in
SQL SELECT queries are not supported.

SQL SELECT queries must be of the following format.

    SELECT [DISTINCT] [table-alias.]column [[AS] alias], ...
      FROM table [[AS] table-alias]
      WHERE [NOT] condition [AND | OR condition] ...
      GROUP BY column ... [HAVING condition ...]
      ORDER BY column [ASC | DESC] ...
      LIMIT n [OFFSET n]

Each column is either a named column, *, a constant value, NULL, CURRENT_DATE,
CURRENT_TIME, or an expression including functions, aggregate functions,
operations +, -, /, *, %, || and parentheses. Supported comparisons in the
optional WHERE clause are <, >, <=, >=, =, !=, <>, NOT, BETWEEN, LIKE, IS NULL,
IN. Use double quotes around table names or column names containing spaces
or other special characters.

| Function	 | Description |
| -----------|-------------|
| COALESCE(N1, N2, ...)	| Returns first expression that is not NULL |
| DAYOFMONTH(D)	| Extracts day of month from date or timestamp D (first day of month is 1) |
| HOUROFDAY(T)	| Extracts hour of day from time or timestamp T |
| LENGTH(S)	| Returns length of string |
| LOWER(S)	| Converts string to lower case |
| LTRIM(S [, T])	| Removes leading characters from S that occur in T |
| MINUTE(T)	| Extracts minute of hour from time or timestamp T |
| MONTH(D)	| Extracts month from date or timestamp D (first month is 1) |
| NULLIF(X, Y)	| Returns NULL if X and Y are equal, otherwise X |
| ROUND(N)	| Rounds N to nearest whole number |
| RTRIM(S, [, T])	| Removes trailing characters from S that occur in T |
| SECOND(T)	| Extracts seconds value from time or timestamp T |
| SUBSTRING(S, N [, L])	| Extracts substring from S starting at index N (counting from 1) with length L |
| TRIM(S, [, T])	| Removes leading and trailing characters from S that occur in T |
| UPPER(S)	| Converts string to upper case |
| YEAR(D)	| Extracts year from date or timestamp D |

Additional functions are defined from java methods using the function.NAME driver property.

| Aggregate Function	| Description |
|-----------------------|-------------|
| AVG(N)	| Average of all values|
| COUNT(N)	| Count of all values|
| MAX(N)	| Maximum value|
| MIN(N)	| Minimum value|
| SUM(N)	| Sum of all values|

For queries containing ORDER BY, all records are read into memory and sorted.
For queries containing GROUP BY plus an aggregate function, all records are read
into memory and grouped. For queries that produce a scrollable result set, all
records up to the furthest accessed record are held into memory. For other queries,
this driver holds only one record at a time in memory.

## Driver Properties

The driver also supports a number of parameters that change the default behaviour of the driver.

These properties are

*charset*

type: String

default: Java default

Defines the character set name of the files being read, such as UTF-16. See the Java Charset documentation for a list of available character set names.

*columnTypes*

type: String

default: all Strings

A comma-separated list defining SQL data types for columns in tables. When column values are fetched using getObject (as opposed to getString), the driver will parse the value and return a correctly typed object. If fewer data types are provided than the number of columns in the table, the last data type is repeated for all remaining columns. If columnTypes is set to an empty string then column types are inferred from the data. When working with multiple tables with different column types, define properties named columnTypes.CATS and columnTypes.DOGS to define different column types for tables CATS and DOGS.

*commentChar*

type: String

default: null

Lines before the header that start with the comment are skipped. After the header has been read, all lines are interpreted as data.

*cryptoFilterClassName*

type: Class

default: null

The full class name of a Java class that decrypts the file being read. The class must implement interface org.relique.io.CryptoFilter. The class org.relique.io.XORFilter included in CsvJdbc implements an XOR encryption filter.

*cryptoFilterParameterTypes*

type: String

default: String

Comma-separated list of data types to pass to the constructor of the decryption class set in property cryptoFilterClassName.

*cryptoFilterParameters*

type: String

default:

Comma-separated list of values to pass to the constructor of the decryption class set in property cryptoFilterClassName.

*defectiveHeaders*

type: Boolean

default: False

in case a column name is the emtpy string, replace it with COLUMNx, where x is the ordinal identifying the column.

*fileExtension*

type: string

default: ".csv"

Specifies file extension of the CSV files. If the extension .dbf is used then files are read as dBase format database files.

*fileTailParts*

type: String

default: null

Comma-separated list of column names for the additional columns generated by regular expression groups in the property fileTailPattern.

*fileTailPattern*

type: String

default: null

Regular expression for matching filenames when property indexedFiles is True. If the regular expression contains groups (surrounded by parentheses) then the value of each group in matching filenames is added as an extra column to each line read from that file. For example, when querying table test, the regular expression -(\d+)-(\d+) will match files test-001-20081112.csv and test-002-20081113.csv. The column values 001 and 20081112 are added to each line read from the first file and 002 and 20081113 are added to each line read from the second file.

*fileTailPrepend*

type: Boolean

default: False

when True, columns generated by regular expression groups in the fileTailPattern property are prepended to the start of each line. When False, the generated columns are appended after the columns read for each line.

*fixedWidths*

type: String

default: null

Defines character position ranges for each column in a fixed width file. When set, column values are extracted from these ranges in each line instead of separating the line by delimiters. Each column is a pair of character positions separated by a minus sign, or a single character for columns with only a single character. The position of the first character on each line is 1. Character position ranges are separated by commas. For example, 1,2-9,16-19.

*function.NAME*

type: String

default: None

Defines a java method to use as the SQL function named NAME in SQL statements. The property value is a public static java given as a java package, class and method name followed by parameter list in parentheses. For example, property function.POW with value java.lang.Math.pow(double, double) makes POW available as an SQL function. Methods with variable length argument lists are defined by appending ... after the last parameter. Each method parameter must be a numeric type, String, or Object.

*headerline*

type: string

default: None

Used in combination with the suppressHeaders property to specify a custom header line for tables. headerline contains a list of column names for tables separated by the separator. When working with multiple tables with different headers, define properties named headerline.CATS and headerline.DOGS to define different header lines for tables CATS and DOGS.

*ignoreNonParseableLines*

type: Boolean

default: False

when True, lines that cannot be parsed will not cause an exception but will be ignored. Each ignored line is logged. Call method java.sql.DriverManager.setLogWriter before executing a query to capture a list of ignored lines.

*indexedFiles*

type: Boolean

default: False

when True, all files with a filename matching the table name plus the regular expression given in property fileTailPattern are read as if they were a single file.

*isHeaderFixedWidth*

type: Boolean

default: True

Used in combination with the fixedWidths property when reading fixed width files to specify whether the header line containing the column names is also fixed width. If False, column names are separated by the separator.

*quotechar*

type: Character

default: "

Defines quote character. Column values surrounded with the quote character are parsed with the quote characters removed. This is useful when values contain the separator or line breaks. No more than one character is allowed. An empty value disables quoting.

*quoteStyle*

type: String

default: SQL

Defines how a quote character is interpreted inside a quoted value. When SQL, a pair of quote characters together is interpreted as a single quote character. When C, a backslash followed by a quote character is interpreted as a single quote character.

*locale*

type: String

default: Java default

Defines locale to use when parsing timestamps. This is important when parsing words such as December which vary depending on the locale. Call method Locale.toString() to convert a locale to a string.

*separator*

type: String

default: ","

Defines column separator. A separator longer than one character is permitted.

*skipLeadingLines*

type: Integer

default: 0

after opening a file, skip this many lines before starting to interpret the contents.

*skipLeadingDataLines*

type: Integer

default: 0

after reading the header from a file, skip this many lines before starting to interpret lines as records.

*suppressHeaders*

type: boolean

default: False

Used to specify that the file does not contain a column header with column names. If True and headerline is not set, then columns are named sequentially COLUMN1, COLUMN2, ... If False, the column header is read from the first line of the file.

*timestampFormat*, *timeFormat*, *dateFormat*

type: String

default: yyyy-MM-dd HH:mm:ss, HH:mm:ss, yyyy-MM-dd

Defines the format from which columns of type Timestamp, Time and Date are parsed. See the Java SimpleDateFormat documentation for date and timestamp patterns.

*timeZoneName*

type: String

default: UTC

The time zone of Timestamp columns. To use the time zone of the computer, set this to the value returned by the method java.util.TimeZone.getDefault().getID().

*trimHeaders*

type: Boolean

default: True

If True, leading and trailing whitespace is trimmed from each column name in the header line. Column names inside quotes are not trimmed.

*trimValues*

type: Boolean

default: False

If True, leading and trailing whitespace is trimmed from each column value in the file. Column values inside quotes are not trimmed.


# Credits

Original authors are

Jonathan Ackerman

Mario Frasca

Sander Brienen

Simon Chenery

# License

JDBC Driver CSV

This library is free software; you can redistribute it and/or modify it under
the terms of the GNU Lesser General Public License as published by the
Free Software Foundation; either version 2.1 of the License, or
(at your option) any later version.

This library is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser General Public License
for more details.

You should have received a copy of the GNU Lesser General Public License
along with this library; if not, write to the Free Software Foundation,
Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307 USA
