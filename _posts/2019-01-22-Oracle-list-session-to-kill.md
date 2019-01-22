---
layout: post
title: "Oracle list sessions to kill"
categories: snippets
tags: [oracle]
comments: true
---

## What is this about?

Sometimes it is useful to kill all active Oracle sessions. The following
snippet let us to list all open sessions and also create statements that
could be run in order to kill them.  

## SQL snippet
<input type="button" value="Copy to Clipboard" onclick="copyToClipboard('pre.highlight > code')"/>

```sql
-- query sessions
SELECT a.username, a.osuser, a.program, sid, a.serial#
FROM v$session a order by username;

-- query session and related process
SELECT a.username, a.osuser, a.program, spid,module, sid, a.serial#
FROM v$session a, v$process b WHERE a.paddr = b.addr order by username;

-- query sessions without any related process
SELECT a.username, a.osuser, a.program, sid, a.serial#
FROM v$session a
WHERE a.paddr NOT IN (select b.addr from v$process b)
ORDER BY username;

-- kill oracle sessio
ALTER SYSTEM KILL SESSION 'a,b';

-- list all sessions to kill
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''';'
FROM v$session;

-- identify sessions that are locking tables (A.OS_User_Name=v$session.username)
SELECT B.Owner, B.Object_Name, A.Oracle_Username, A.OS_User_Name
FROM V$Locked_Object A, All_Objects B
WHERE A.Object_ID = B.Object_ID
```
