[octo]() - A GCC C Library
========================================
[![Build Status](https://drone.io/github.com/shrimpboyho/octo/status.png)](https://drone.io/github.com/shrimpboyho/octo/latest)

A library for doing various things in C

How to Install
--------------

First get a copy of the library by git cloning it:

```bash
git clone https://github.com/shrimpboyho/octo.git
cd octo
```

Then you can just include the header file in your code:

```c
#include "octo.h"
```

Documentation Regarding Strings
-------------------------------
###Get info on a string

```c
size_t len(char* thing);
```
	
Returns the number of characters in ```thing``` excluding the null character.

```c
char charAt(char* s, int n);
```

Returns the character in the string ```s``` at the index of ```n```

###Make an identical copy of a string

```c
char* makeCopy(char* thing);
```

Creates a new copy of ```thing``` in the memory and returns a pointer to it.

###Modify the original string

```c
void* strip(char* thing);
```
	
Trims off both leading and trailing whitespace on ```thing```. It modifies the original buffer.

```c
void* lstrip(char* thing);
```
	
Trims off only the leading whitespace on ```thing```. It modifies the original buffer.

```c
void* rstrip(char* thing);
```
	
Trims off only the trailing whitespace on ```thing```. It modifies the original buffer.

###Create new versions of strings

```c
char* slice(char* thing, int start, int end, int BYTES);
```
	
Creates a new string from ```thing``` that consists of the characters from the starting index (```start```), to the ending index (```end```), and returns a ```char*``` to it. It does not modify the original buffer. The argument ```BYTES``` is usually zero if you want the returned string to perfectly fit in the buffer. If you plan on modifying the string after you get it from this function, it is best to set ```BYTES``` to some other value, such as 30, giving you 30 bytes of additional padding.
```c
char* replaceBetween(char* s, int start, int end, char* ham);
```

Replaces the content of the string ```s``` between the indexes of ```start``` and ```end``` with the content of the string ```ham```. Returns a pointer to the new buffer.

```c
char* toUpperCase(char* s);
```

Creates a new copy of ```s``` in which all the characters are uppercase and returns a pointer to it.

```c
char* toLowerCase(char* s);
```

Creates a new copy of ```s``` in which all the characters are lowercase and returns a pointer to it.


###Convert between strings and other data types

```c
int stringToInt(char* s);
```

Converts ```s``` to an integer and returns it.

```c
char* intToString(int n);
```

Converts ```n``` to a string and returns a pointer to it.

Documentation Regarding WUMBO
-----------------------------------------

```wumbo``` is the simple math expression parser that comes packaged with ```octo```. ```wumbo``` parses a math expression contained within a string and returns the result as a double. The following code outlines how it is generally used. Note that since this module is not complete, although it functions, there may be memory leaks.

```c
#include <stdio.h>
#include <string.h>
#include "octo.h"

int main( void ){

    double answer = WUMBO_parse("2 + ( 2 / 3 )");
    printf("\nAnswer: %lf\n", answer);

    return 0;
}
```

Documentation Regarding The JSON Database
-----------------------------------------
The following code shows how the database API basically works.

```c
#include "database.h"
#include <stdlib.h>
#include <stdio.h>

/* Define how greedy you want octo to be with memory */
#define DEPTH 20

int main ( void )
{

    /* Create a database */
    DB *bank = newDB("Employees");
    printf("Name of db is: %s", bank -> name);
    addValue(bank, "Daniel", "Programmer", STRING_TYPE);
    addValue(bank, "Kevin", "Hardware", STRING_TYPE);
    
    /* Create another database*/
    DB *money = newDB("Money");
    addValue(money, "Monday", "5", NUM_TYPE);
    addValue(money, "Tuesday", "7", NUM_TYPE);

    /* Shove the money database in the bank */
    addDB(bank,money);

    /* Pretty print the entire database*/
    prettyPrint(bank);

    /* Dump database contents to json file */
    dumpDB(bank, "bank.json");
    
    return 0;
}
```

```octo``` currently has very limited JSON database support and functionality. For now, one can dump JSON contents to a file after building the file via the API. Note that within ```octo``` the term ```DB``` (database) is synonymous with the term ```JSON Document```

####A Word on Memory

```octo``` uses ```malloc()``` and ```free()``` to create format strings of variable size, for the purposes of printing in a nice formatted manner. The reason we do this is so that JSON within JSON can be properly tabulated. This means that we must build up a format string with a lot of tabs depending on how much depth a JSON file has.

There is one constant known as ```DEPTH``` which is a number. By default it is 20 bytes. It is recommended that ```DEPTH``` be at least 20 bytes. However, one can redefine it to be bigger.

```c
#define DEPTH 2323
```

####Simple Database Creation

Creating a database is simple and can be done by invoking the following function:

```c
/* Create an empty database */
DB *bank = newDB("Employees");
```

The JSON document is represented as such:

```JSON
{
    "Employees" : {
    
    }
}
```

From here one can access the JSON document name:

```c
char *dbname = bank -> name;
printf("Name of db is: %s", dbname);

/** OUTPUT:
*	Name of db is: Employees
*/
```

####Adding Data To A Database

Adding a simple key value pair can be done by calling the following function:

```c
DB *money = newDB("Money");
addValue(money, "Monday", "5", NUM_TYPE);
```
The above code adds the value of ```5``` with the key of ```"Monday"``` to the database known as ```money```. Notice the fourth argument is a constant that specifies that the value of ```5``` is to be interpretted as a number rather than as a string.

There are several type constants you can use as arguments to the ```addValue()``` function:

```c
NUM_TYPE /* For all types of numbers, decimal and floating point */
STRING_TYPE /* For strings */
BOOL_TYPE /* For true and false booleans */
ARRAY_TYPE /* For array types */
```

####Adding JSON inside of JSON

This is as simple as adding a database to another database. The ```addDB()``` function takes two arguments: the destination database and the source database.

```c
/* Create a database */
DB *bank = newDB("Employees");

/* Create another database*/
DB *money = newDB("Money");

/* Shove the money database in the bank database */
addDB(bank, money);
```

####Deleting Stuff

Anything within a database can be deleted if you know the key. The following code shows how the value associated with the key of ```Nash``` can be deleted from the database.

```c
deleteByID(bank, "Nash");
```

Note that you can also delete an entire database within a database in the same manner. For example, the following code removes the database known as ```Money```:

```c
deleteByID(bank, "Money");
```
