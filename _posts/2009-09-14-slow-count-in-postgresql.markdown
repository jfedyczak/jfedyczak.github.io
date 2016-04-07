---
layout: post
title: "Slow COUNT(*) in PostgreSQL"
date: 2009-09-14
tags: slow count, postgresql
permalink: /post/slow-count-in-postgresql/
uid: 53B95320-46F2-4656-9C9B-17DE2956E707
---
`COUNT(*)` in PostgreSQL tends to be slow. I takes about 20 secodns to count 1.7 million records table. I's not a bug. It's a feature of [MVCC](http://en.wikipedia.org/wiki/Multiversion_concurrency_control). One of the workarounds of the problem is a row counting trigger with a helper table:

{% highlight sql %}
begin;

create table table_count(
        table_count_id text primary key,
        rows int default 0
);

CREATE OR REPLACE FUNCTION table_count_update()
RETURNS trigger AS
$BODY$
begin
    if tg_op = 'INSERT' then
        update table_count set rows = rows + 1 
            where table_count_id = TG_TABLE_NAME;
    elsif tg_op = 'DELETE' then
        update table_count set rows = rows - 1 
            where table_count_id = TG_TABLE_NAME;
    end if;
    return null;
end;
$BODY$
LANGUAGE 'plpgsql' VOLATILE;

commit;
{% endhighlight %}

Next step is to add proper trigger declaration for each table you'd like to use it with. For example for table `tab_name`:

{% highlight sql %}
begin;
insert into table_count values 
    ('tab_name',(select count(*) from tab_name));

create trigger tab_name_table_count after insert or delete
on tab_name for each row execute procedure table_count_update();
commit;
{% endhighlight %}
    
It is important to run in a transaction block to keep actual count and helper table in sync in case of delete or insert between initial count and trigger creation. Transaction guarantees this. From now on to get current count instantly, just invoke:

{% highlight sql %}
select rows from table_count where table_count_id = 'tab_name';
{% endhighlight %}
    
That's it. Now you have super fast `COUNT(*)` for your tables.
