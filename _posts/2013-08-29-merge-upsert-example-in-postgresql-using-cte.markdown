---
layout: post
title: "MERGE/UPSERT example in PostgreSQL using CTE"
date: 2013-08-29
tags: merge, upsert, postgresql, cte
permalink: /post/merge-upsert-example-in-postgresql-using-cte/
uid: 7436AFEF-9BA1-4E55-B2D3-49F90917164A
---
## Introduction

PostgreSQL doesn't provide direct MERGE or UPSERT commands. But similar results can by achieved using [CTE](http://www.postgresql.org/docs/9.2/interactive/queries-with.html) (Common Table Expressions).

I'll use following table to illustrate this solution:

{% highlight sql %}
CREATE TABLE person (
    name TEXT PRIMARY KEY,
    age INT NOT NULL
);

INSERT INTO person VALUES
    ('John', 31),
    ('Jane', 24);
{% endhighlight %}

## INSERT not existing rows

I'll now try to insert some new records (one new and one already existing name):

{% highlight sql %}
WITH
    to_be_inserted (name, age) AS (
        VALUES
            ('John', 46),
            ('Carol', 37)
    ),
    existing AS (
        SELECT
            name
        FROM
            to_be_inserted
        WHERE
            EXISTS (
                SELECT 1 FROM person
                WHERE name = to_be_inserted.name
            )
    )
INSERT INTO person
    SELECT * FROM to_be_inserted
    WHERE name NOT IN (SELECT name FROM existing)
    RETURNING *;
{% endhighlight %}

This yields:

     name  | age 
    -------+-----
     Carol |  37
    (1 row)

    INSERT 0 1

And after issuing `SELECT * FROM person;` we'll get:

     name  | age 
    -------+-----
     John  |  31
     Jane  |  24
     Carol |  37
    (3 rows)

In this `WITH` query I defined `to_be_inserted` as static collection of rows to be inserted, `existing` collection as names that are present in `to_be_inserted` and exists in `person` table and then we use both these collections to perform conditional `INSERT`. All in one query.

## Updating exitsing rows and inserting new ones

Now I'll try to update two records and insert non-existing one:

{% highlight sql %}
WITH
    to_be_upserted (name, age) AS (
        VALUES
            ('John', 46),
            ('Carol', 23),
            ('Edgar', 18)
    ),
    updated AS (
        UPDATE
            person
        SET
            age = to_be_upserted.age
        FROM
            to_be_upserted
        WHERE
            person.name = to_be_upserted.name
        RETURNING person.name
    )
INSERT INTO person
    SELECT * FROM to_be_upserted
    WHERE name NOT IN (SELECT name FROM updated);
{% endhighlight %}

And after issuing `SELECT * FROM person;` again we'll get:

     name  | age 
    -------+-----
     Jane  |  24
     John  |  46
     Carol |  23
     Edgar |  18

Here I defined `to_be_upserted` static collection and used it to define `updated` collection which returns only updated (i.e. existing) rows and then I issue `INSERT` query to insert non-existent rows.

Using CTEs with data-modifying statements can save some round trips between database and application.
