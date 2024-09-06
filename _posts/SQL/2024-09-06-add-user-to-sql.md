---
layout: post
title:  "Add User in SQL"
description: "Add authenticated users that can be used by applications"
date: 2024-09-06
tags: sql user login authentication
category: SQL
---
## Add SQL User

### Add User in the Master DB

```sql
CREATE LOGIN user WITH PASSWORD = '**pass**';
```

### Add User in the Working DB

```sql
CREATE USER user FOR LOGIN user
```

### Grant CRUD Access to the User in the Working DB

```sql
ALTER ROLE db_datareader ADD MEMBER user
ALTER ROLE db_datawriter ADD MEMBER user
```
