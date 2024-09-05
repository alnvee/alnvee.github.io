---
layout: post
title:  "Working with JSON in SQL Stored Procedures"
date: 2024-09-05
tags: sql procs stored procedures json
category: SQL
---
## Working with JSON in SQL Stored Procedures

Save Accounts Colors to a temporary table @accountColorsTable

```sql
DECLARE @accountColorsTable TABLE(
    id NVARCHAR(32),
    hex NVARCHAR(7),
    name NVARCHAR(200)
);

INSERT INTO @accountColorsTable (id, hex, name)
SELECT *
FROM OPENJSON(@accountColorsJSON)
WITH(
    id NVARCHAR(32) '$.id',
    hex NVARCHAR(7) '$.hex',
    name NVARCHAR(32) '$.name'
);
```

\
-Insert to Projects table if code(workspace_id) does not exist \
-Update if code exists

```sql
MERGE Projects AS tgt
USING (
    SELECT
        JSON_VALUE(JsonElement, '$.title') AS title,
        JSON_VALUE(JsonElement, '$.workspace_id') AS workspace_id,
        SWITCHOFFSET(CONVERT(DATETIMEOFFSET, JSON_VALUE(JsonElement, '$.updated_at'), 127), DATEPART(TZOFFSET, SYSDATETIMEOFFSET()))  AS updated_at --convert to server date
    FROM (
        SELECT [value] AS JsonElement
        FROM OPENJSON(@projectsJSON)
    ) ParsedJson
) AS src(
        Name, 
        WorkspaceID, 
        UpdatedAt
)
ON tgt.Code = src.WorkspaceID 
WHEN MATCHED 
AND tgt.ModifyDate <= src.UpdatedAt
THEN
    UPDATE
    SET Code = src.WorkspaceID,
        Name = src.Name,
        ModifyDate = GETDATE()
WHEN NOT MATCHED
THEN
    INSERT (
        Code, 
        Name, 
        CreateDate
    )
    VALUES (
        src.WorkspaceID,
        src.Name, 
        (SELECT name FROM @accountColorsTable WHERE id = src.AccountColorID),
        'System',
        GETDATE()
    );
```
