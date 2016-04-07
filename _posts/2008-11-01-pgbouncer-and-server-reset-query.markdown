---
layout: post
title: "pgBouncer and server_reset_query"
date: 2008-11-01
tags: pgBouncer, server_reset_query, DISCARD ALL
permalink: /post/pgbouncer-and-server-reset-query/
uid: ECD9B84D-DDBE-4AD0-973F-FCB16039FB4B
---
**Update**: As of PostgreSQL 8.3.6 `DISCARD ALL` do release all advisory locks. This makes part of this article obsolete.

I love pgBouncer. It is state of the art PostgreSQL database connection pooler. It is so good, that you can upgrade it without interrupting single connection!

In its most conservative mode connection is reused. This eliminates substantial time waste needed to establish "real" connection do pgsql server. I've managed to reduce CPU load on a busy PHP-based site by the factor of 10 (ten!).

When client disconnects from pgBouncer, real database connection has to be reset before making it available to another client. To accomplish this, pgBounces issues `DISCARD ALL` command on this connection. 

There are two problems with `DISCARD ALL`:

1. It doesn't work within a transaction.
2. It doesn't unlock any of pgsql's advisory locks.

Imagine that a script issues `BEGIN` and than dies. The connection to pgBouncer closes and pgBouncer issues `DISCARD ALL`, which doesn't do anything because of active transaction. A new client connects and uses the same connection. Every command it issues fails because of *"commands ignored to the end of transaction"*. Not good.

Now imagine that script issues `SELECT pg_advisory_lock(xxx)` and then dies unexpectedly. `DISCARD ALL` issued by pgBouncer after that does nothing with it. Another script on another "real" connection issues `SELECT pg_advisory_lock(xxx)` and locks forever because there is no one to call `SELECT pg_advisory_unlock(xxx);` on previous connection.

My solution covers both issues - change `server_reset_query` setting in pgbouncer.ini:

    server_reset_query =
        ROLLBACK; SELECT pg_advisory_unlock_all(); DISCARD ALL;

This ensures, that any transaction left from faulty code will be closed and rolled back and every lock left will be released.

Have fun with pgBouncer!
