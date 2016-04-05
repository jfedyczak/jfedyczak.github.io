---
layout: post
title: "Regexps in CHECK constraints"
date: 2008-12-09
tags: regexp, check, postgresql
permalink: /post/regexps-in-check-constraints/
---
Regexps (regular expressions) are very powerful and robust tool to use in PostgreSQL. You're probably familiar with the syntax:

    SELECT * FROM record WHERE extra_field ~ 'AX[0-9]{3}B';
    
My tip for today is - use regexp with `CHECK` column constraints! For example:

    CREATE TABLE employee (
        employee_id SERIAL PRIMARY KEY,
        login TEXT NOT NULL UNIQUE
            CHECK (login ~ '^[-_0-9a-zA-Z]{3,}$')
    );
    
This will ensure that login will consist only of alphanumerical characters, hyphens and/or underscores and it cannot be shorter than 3 characters. I love it!