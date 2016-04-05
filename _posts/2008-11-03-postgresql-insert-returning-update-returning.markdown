---
layout: post
title: "PostgreSQL's INSERT ... RETURNING and UPDATE ... RETURNING"
date: 2008-11-03
tags: postgres, postgresql, insert returning, update returning
permalink: /post/postgresql-insert-returning-update-returning/
---
I've just discovered best thing since transactions in PosrgreSQL. Let's try to create and populate a table:

{% highlight sql %}
CREATE TABLE test (test_id SERIAL PRIMARY KEY, a INT NOT NULL);
INSERT INTO test (a) VALUES
    (1), (2), (3), (4), (5),
    (6), (7), (8), (9), (10);
SELECT * FROM test;
{% endhighlight %}
This gives us:

     test_id | a  
    ---------+----
           1 |  1
           2 |  2
           3 |  3
           4 |  4
           5 |  5
           6 |  6
           7 |  7
           8 |  8
           9 |  9
          10 | 10
Now let's say we want to know the value of `test_id` column after another `INSERT` - this is where interesting stuff begins:

{% highlight sql %}
INSERT INTO test (a) VALUES (50) RETURNING *;
{% endhighlight %}

     test_id | a  
    ---------+----
          11 | 50

Notice that `RETURNING` clause acts like SELECT over newly inserted fields. To obtain value of `SERIAL` column in another way, you would have to get current value of `test_test_id_seq` discrete sequence and you'd have to put it inside transaction block to be sure it's accurate. This one is MUCH simpler.

Another trick with `RETURNING` is possible with `UPDATE`:

{% highlight sql %}
UPDATE test SET a = a + 100 WHERE a < 6 RETURNING *;
{% endhighlight %}

     test_id |  a  
    ---------+-----
           1 | 101
           2 | 102
           3 | 103
           4 | 104
           5 | 105

This results in a set of rows altered by `UPDATE`. Now you know how many records have you changed and in what way. Try to accomplish this without `RETURNING`...

You can also narrow result by listing column explicitly rather than using `*`. 
