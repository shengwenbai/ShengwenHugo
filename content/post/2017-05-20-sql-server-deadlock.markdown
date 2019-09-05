---
layout:     post
title:      "SQL Server deadlocks"
subtitle:   ""
date:       2016-12-21 12:00:00
author:     "Shengwen"
header-img: "img/database-bg.jpg"
tags:
    - SQL Server
    - ADO.NET
---

### Catalog
1. [How SQL Server detects deadlock?](#cat0)
2. [Deadlock Priority](#cat1)
3. [What is the deadlock victim selection criteria?](#cat2)
4. [Logging deadlock](#cat3)
5. [Handling deadlocks in ado.net](#cat4)


<p id="cat0"/>
### How SQL Server detects deadlock?
Lock monitor thread in SQL Server, runs every 5 seconds by default to detect if there are any deadlocks.  If the lock monitor thread finds deadlocks, the deadlock detection interval will drop from 5 seconds to as low as 100 milliseconds depending on the frequency of deadlocks. When a deadlock is detected, the Database Engine ends the deadlock by choosing one of the threads as the deadlock victim. The deadlock victim's transaction is then rolled back and returns a 1205 error to the application. 
![](/img/in-post/post-concurrent/deadlock.png)

<p id="cat1"/>
### Deadlock Priority
1. The default is Normal
2. Can be set to LOW, NORMAL, or HIGH
3. Can also be set to a integer value in the range of -10 to 10.
LOW : -5
NORMAL : 0
HIGH : 5 

<p id="cat2"/>
### What is the deadlock victim selection criteria?
1. If the DEADLOCK_PRIORITY is different, the session with the lowest priority is selected as the victim
2. If both the sessions have the same priority, the transaction that is least expensive to rollback is selected as the victim
3. If both the sessions have the same deadlock priority and the same cost, a victim is chosen randomly 

<p id="cat3"/>
### Logging deadlock
Use SQL Server trace flag 1222 to write the deadlock information to the SQL Server error log. 
Enable Trace flag : To enable trace flags use DBCC command. -1 parameter indicates that the trace flag must be set at the global level. If you omit -1 parameter the trace flag will be set only at the session level. 
```sql
DBCC Traceon(1222, -1)
```
To check the status of the trace flag
```sql
DBCC TraceStatus(1222, -1)
```
To turn off the trace flag
```sql
DBCC Traceoff(1222, -1)
```
To read the error log
```sql
execute sp_readerrorlog
```
Deadlock error handling
```sql
Begin Catch
    --Check if the error is deadlock error
    If(ERROR_NUMBER() = 1205)
    Begin
        Select 'Deadlock. Transaction failed. Please retry'
    End
    --Rollback the transaction
    Rollback
End Catch
```

<p id="cat4"/>
### Handling deadlocks in ado.net
```csharp
catch (SqlException ex)
{
    if (ex.Number == 1205)
    {
        Label1.Text = "Deadlock. Please retry";
    }
    else
    {
        Label1.Text = ex.Message;
    }
    Label1.ForeColor = System.Drawing.Color.Red;
}
```

<p id="cat5"/>
### Retry logic for deadlock exceptions
When a transaction fails due to deadlock, we can write some logic so the system can resubmit the transaction. The deadlocks usually last for a very short duration. So upon resubmitting the transaction it may complete successfully. 


Reference: [kudvenkat's YouTube channel](https://www.youtube.com/user/kudvenkat)
