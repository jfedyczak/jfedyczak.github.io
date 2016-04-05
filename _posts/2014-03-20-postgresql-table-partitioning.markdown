---
layout: post
title: "PostgreSQL table partitioning"
date: 2014-03-20
tags: postgresql, partitioning
permalink: /post/postgresql-table-partitioning/
---
When you need to store large amount of indexed data in one table, you hit the problem of slow inserts caused by large [B-tree](http://en.wikipedia.org/wiki/B-tree) needing to be balanced every time you add data. PostgreSQL provides mechanism allowing to split data across multiple sub-tables. This reduces index size considerably and makes speed of inserting new data more consistent.

## Introduction

As per documentation, partitioning should be considered in cases where amount of data in one table exceeds amount of RAM available.

The generali approach is to create master table without any constraints and indexes. For example:

{% highlight sql %}
CREATE TABLE temperature (
    ts timestamp(0) not null default now(),
    c int
);
{% endhighlight %}

The next step is to create sub-tables holding real data. The key to success is the [INHERITS](http://www.postgresql.org/docs/9.3/interactive/sql-createtable.html) keyword. It creates relation between child and parent table allowing database to gather data from all child tables as one result:

{% highlight sql %}
CREATE TABLE temperature_20140320 INHERITS (temperature);
ALTER TABLE temperature_20140320 ADD CONSTRAINT ts_check CHECK (ts >= '2014-03-20' AND ts < '2014-03-21');
CREATE INDEX temperature_20140320_ts_idx ON temperature_20140320(ts);
{% endhighlight %}

By creating multiple tables like this you effectively create partitioned table. Issuing:

{% highlight sql %}
SELECT * FROM temperature;
{% endhighlight %}

returns all data from all child tables (created with `INHERITS`). It is important to create CHECK constraints as query planner takes them into consideration when selecting child tables as data sources.

Also remember not to insert values directly into master table. Place appropriate data in relevant child tables.

## Automating the process

It is easy to make data insertions and child table creation process automatic with a trigger function:

{% highlight sql %}
CREATE OR REPLACE FUNCTION l_insert() RETURNS trigger AS
$BODY$
    DECLARE
        _name text;
        _from timestamp(0);
        _to timestamp(0);
    BEGIN
        SELECT into _name 'temperature_'||replace(new.ts::date::text,'-','');
        IF NOT EXISTS (SELECT 1 FROM information_schema.tables WHERE table_name=_name)  then
            SELECT into _from new.ts::date::timestamp(0);
            SELECT into _to _from + INTERVAL '1 day';
            EXECUTE 'CREATE TABLE '||_name||' () INHERITS (temperature)';
            EXECUTE 'ALTER TABLE '||_name||' ADD CONSTRAINT ts_check CHECK (ts >= '||quote_literal(_from)||' AND ts < '||quote_literal(_to)||')';
            EXECUTE 'CREATE INDEX '||_name||'_ts_idx on '||_name||'(ts)';
        END IF;
        EXECUTE 'INSERT INTO '||_name||' (ts,c) VALUES ($1,$2)' USING
            new.ts,new.c;
        RETURN null;
    END;
$BODY$
  LANGUAGE plpgsql;
{% endhighlight %}

And a trigger:

{% highlight sql %}
CREATE TRIGGER temperature_insert
    BEFORE INSERT
    ON temperature
    FOR EACH ROW
    EXECUTE PROCEDURE temperature_insert();
{% endhighlight %}

This creates the trigger responsible for automatic creation of child tables with all necessary constraints and indexes. With this trigger you can perform `INSERT` statements directly on master table.
