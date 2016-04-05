---
layout: post
title: "PostgreSQL WITH RECURSIVE tree traversing example"
date: 2012-10-30
tags: WITH RECURSIVE, PosrgreSQL
permalink: /post/postgresql-with-recursive-tree-traversing-example/
---
Here's complete example of traversing a tree in depth-first manner (ensured by `ORDER BY` clause).

{% highlight sql %}
create table empl (
    name text primary key,
    boss text null
        references name 
            on update cascade 
            on delete cascade 
        default null
);

insert into empl values ('Paul',null);
insert into empl values ('Luke','Paul');
insert into empl values ('Kate','Paul');
insert into empl values ('Marge','Kate');
insert into empl values ('Edith','Kate');
insert into empl values ('Pam','Kate');
insert into empl values ('Carol','Luke');
insert into empl values ('John','Luke');
insert into empl values ('Jack','Carol');
insert into empl values ('Alex','Carol');

with recursive t(level,path,boss,name) as (
        select 0,name,boss,name from empl where boss is null
    union
        select
            level + 1,
            path || ' > ' || empl.name,
            empl.boss,
            empl.name 
        from 
            empl join t 
                on empl.boss = t.name
) select * from t order by path;
{% endhighlight %}
Hope it helps someone - took me a while to figure it out.
