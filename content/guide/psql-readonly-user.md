---
title: How to Create a Read-Only User in PostgreSQL?
draft: false
date: 2025-05-31
tags:
  - postgresql
  - database
---

Sometimes, a user with limited permissions is needed, only to read and get metrics. For such cases, you can use the following SQL code:

```sql
-- Creating a user
CREATE USER <USER> WITH PASSWORD <PASSWORD>;

-- Granting connection permission to the database
GRANT CONNECT ON DATABASE <DATABASE> TO <USER>;

-- Setting readonly permissions in the database
\c <DATABASE>
GRANT USAGE ON SCHEMA public TO <USER>;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO <USER>;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO <USER>;
```

SQL

This gives SELECT permission and connection permission to all existing tables in the public schema of the database. It also automatically assigns SELECT permission for tables to be created in the future.
