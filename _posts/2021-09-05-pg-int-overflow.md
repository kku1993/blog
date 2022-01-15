---
layout: post
title:  "No Integer Wrap Around in Postgres"
date:   2021-09-05
categories: programming, database, postgres
---

Integers in Postgres do not wrap around. Instead they throw an "integer out of
range" error.

I had an application that stored a counter in postgres. The counter was used as
a 3 digit ticket number that just cycles through (e.g. 000 to 999 then back to
000). To save space, I used a `smallint` with explicit logic to handle
overflows / wrap around in my application. 

Unfortunately, one day the counter just refuses to get incremented anymore
because of the overflow :(
