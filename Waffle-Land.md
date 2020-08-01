# Waffle Land

### Points: 150

### Description

![Description](https://github.com/alph4byt3/CTF-Stuff/blob/master/images/description.png)

## Solution

After opening the link, I was given a simple webpage with a few products shown along with a search bar and a location to sign in.

![Home Page](https://github.com/alph4byt3/CTF-Stuff/blob/master/images/home.png)  

By placing a single ' in the search bar, I recieved an SQL Error. The error is kind enough to tell me the query used along with the table name of 'product' and the column name of 'name'

I will use these names later to test for a UNION attack.

![SQL error](https://github.com/alph4byt3/CTF-Stuff/blob/master/images/sqlerror.png)

By using the following payload, we can initially determine if the page is vulnerable to a UNION based attack.

> `' ORDER BY 1--`

If the page returns no error, this is a strong indication that the site is vulnerable.

I then start incrementing the value to determine the amount of columns in the product table  


> `' ORDER BY 2`  

> `' ORDER BY 3`  

> `' ORDER BY 4`  

> `' ORDER BY 5`  


Everything until this point shouldn't return any errors but the order of products change due to the query. Once I hit 6, I once again get an error meaning that there are 5 columns in this table.

Knowing this, I can start building the payload.

The first thing I tried is:

> `' UNION SELECT 1,null,null,null,5 FROM product--`

This will cause a 403 response indicating that there is some sort of filtering on the server however this can be bypassed by substituting spaces with comments in the payload.

> `'/**/UNION/**/SELECT/**/1,null,null,null,5/**/FROM/**/product--`

This should add a bunch of products with null values, if we substitute one of the null values with 'name' and 'description', we can populate them.

![UNION](https://github.com/alph4byt3/CTF-Stuff/blob/master/images/union.png)

With this being succesful, the next thing I tried was to dump the table names using the following...

> `'/**/UNION/**/SELECT/**/1,table_name,null,null,5/**/FROM/**/information_schema.tables--`

This returned back a 403 response which made me believe that some keywords were being filtered. After a bunch of attempts of trying to obfuscate the payload even more such as using several encoding methods, comments and techniques found online, non of them would work. It was at this point that I was about to take a break and try later but then I started to wonder if the table and column names I needed are easy and predictable.

After several attempts using common enough words, I was finally able to guess the names correctly and extract the admin username and password using the following payload:

> `'/**/UNION/**/SELECT/**/1,username,password,null,5/**/FROM/**/user--`

![Admin Credentials](https://github.com/alph4byt3/CTF-Stuff/blob/master/images/creds.png)

By navigating to the sign in page and logging in as admin, I was presented with the flag.

![Flag](https://github.com/alph4byt3/CTF-Stuff/blob/master/images/flag.png)

P.S I wasn't really happy that I correctly guessed the names because I do believe that there was a way to correctly bypass the WAF and dump them but I guess the creators of the challenge was kind enough to not name them something fancy :) (also if you're wondering, yes...I correctly guessed the description column name as well xD)
