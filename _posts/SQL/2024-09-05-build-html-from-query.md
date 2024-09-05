---
layout: post
title:  "Build HTML from SQL Query"
description: "Using Stored Procedures to build HTML tables"
date: 2024-09-05
tags: sql procs stored procedures html table
category: SQL
---
## Build HTML from SQL Query

### Generate  HTML table containing query information

```sql
SELECT 
    pc.ProjectId,
    p.Name AS ProjectName,
    p.CustomerName,
    pc.ChecklistName,
    '<table>' +
        '<tr>' +
            '<th>#</th>'+
            '<th>Category</th>'+
            '<th>Activity</th>'+
            '<th>Owner</th>'+
            '<th>Group</th>'+
            '<th>When</th>'+
        '</tr>' +
    STRING_AGG(
        CONCAT(
        '<tr>',
            '<td>',pc.ActivityNumber,'</td>',
            '<td>',pc.Category,'</td>', 
            '<td>',pc.ActivityName,'</td>',
            '<td>',pc.Owner,'</td>',
            '<td>',pc.GroupName,'</td>',
            '<td>',pc.Requirement,'</td>',
        '</tr>'
            ), ''
    ) WITHIN GROUP (ORDER BY [ActivityNumber])
    + '</table>' AS HTMLActivity
FROM CL_ProjectChecklists AS pc
JOIN Projects AS p ON pc.ProjectId = p.ID
WHERE pc.ActivityIsComplete = N'No'
AND pc.ChecklistName <> N'Project Maintaining'
GROUP BY p.CustomerName, p.Name, pc.ProjectId, pc.ChecklistName
```

1. The query selects data from two tables: CL_ProjectChecklists (aliased as pc) and Projects (aliased as p).
2. It joins these tables on the ProjectId column.
3. The main columns selected are:

   - pc.ProjectId
   - p.Name (aliased as ProjectName)
   - p.CustomerName
   - pc.ChecklistName

4. The most complex part is the creation of an HTML table as a string:

   - It then uses STRING_AGG to concatenate rows of data into table rows.
   - Each row contains: ActivityNumber, Category, ActivityName, Owner, GroupName, and Requirement.
   - The WITHIN GROUP (ORDER BY [ActivityNumber]) clause ensures the rows are ordered by ActivityNumber

5. The WHERE clause filters the results:

   - Only includes activities that are not complete (ActivityIsComplete = N'No')
   - Excludes checklists named 'Project Maintaining'

6. The GROUP BY clause groups the results by CustomerName, ProjectName, ProjectId, and ChecklistName. This is necessary because of the use of STRING_AGG, which is an aggregate function.

In summary, this query creates an HTML table for each project, containing all incomplete checklist items (except those in 'Project Maintaining' checklists). The table includes details about each activity, and the activities are ordered by their number within each project.

### Generate an HTML table of with CTE in SQL

```sql
;WITH MainQueryResult AS (
SELECT TOP 100 PERCENT
    pc.ProjectId,
    p.Name AS ProjectName,
    p.CustomerName,
    pc.ChecklistName,
    pc.ActivityNumber,
    pc.Category,
    pc.ActivityName,
    pc.Owner, 
    pc.GroupName,
    pc.Requirement,
    mc.Year,
    mc.Month
FROM CL_MaintainingChecklistActivities AS mc
JOIN CL_ProjectChecklists AS pc ON mc.ProjectChecklistId = pc.ID
JOIN Projects AS p ON pc.ProjectId = p.ID
WHERE mc.ActivityIsComplete = N'No'
    AND pc.ChecklistName = N'Project Maintaining'
)
SELECT 
ProjectId,
ProjectName,
CustomerName,
ChecklistName,
    
    '<table>' +
        '<tr>' +
            '<th colspan=6 style="text-align: center;"  > Period: ' + CONVERT(varchar(4), Year) + '-' + CONVERT(varchar(2), Month) + '</th>' +
        '</tr>' +
        '<tr>' +
            '<th>#</th>'+
            '<th>Category</th>'+
            '<th>Activity</th>'+
            '<th>Owner</th>'+
            '<th>Group</th>'+
            '<th>When</th>'+
        '</tr>' +
    STRING_AGG(
        CONCAT(
            '<tr>',
            '<td>', ActivityNumber, '</td>',
            '<td>', Category, '</td>', 
            '<td>', ActivityName, '</td>',
            '<td>', Owner, '</td>',
            '<td>', GroupName, '</td>',
            '<td>', Requirement, '</td>',
            '</tr>'
        ), ''
    ) WITHIN GROUP (ORDER BY [ActivityNumber])
    + '</table>' AS HTMLActivity
FROM MainQueryResult
GROUP BY CustomerName, ProjectName, ProjectId, ChecklistName, Year, Month
ORDER BY Year DESC, Month DESC,  MIN(ActivityNumber) ASC 
```

1. Common Table Expression (CTE):
The query starts with a CTE named MainQueryResult. This CTE joins three tables:

   - CL_MaintainingChecklistActivities (aliased as mc)
   - CL_ProjectChecklists (aliased as pc)
   - Projects (aliased as p)
   - It selects various columns from these tables, including project details, checklist information, and activity specifics. The CTE filters for activities that are not complete (ActivityIsComplete = 'No') and belong to the 'Project Maintaining' checklist.

2. Main Query:
The main query uses the MainQueryResult CTE to generate an HTML table for each project's incomplete activities.
Key points:

   - It groups the results by project, customer, checklist, year, and month.
   - It uses **STRING_AGG** to concatenate HTML table rows for each activity.
   - The CONCAT function is used to create individual table rows with activity details.
   - The WITHIN GROUP (ORDER BY [ActivityNumber]) clause ensures activities are ordered by their number.

3. HTML Generation:

   - The query creates a table header with the period (year and month).
   - It then adds a row for column headers.
   - Activity details are added as table rows.
   - The entire HTML string is aliased as HTMLActivity.

4. Ordering:
The final results are ordered by year (descending), month (descending), and the minimum activity number (ascending).

This query is useful for generating a report of incomplete project maintaining activities, formatted as an HTML table. Each project will have its own table, showing activities that need attention, along with relevant details like category, owner, and group.
