---
layout: post
title:  "Add User in SQL"
description: "Add authenticated users that can be used by applications"
date: 2024-09-06
tags: sql user login authentication
category: SQL
---
## Add SQL User

### Add User in the Master DB with Password Auth

```sql
CREATE LOGIN user WITH PASSWORD = '**pass**';
```

### Add User in the Master DB using EntraID

```sql
CREATE LOGIN [user@company.com] FROM EXTERNAL PROVIDER
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

### Grant Execute Permissions to Procs in a DB

```sql
USE [DB] GRANT EXEC TO [User_Name];
```
