---
layout: post
title: Re-purposing SQL Server PARSENAME For Splitting IPv4 Addresses
date: 2011-01-20 12:49:43.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Coding
- Random
- SQL
tags:
- charindex
- convert
- ipv4
- parsename
- split
- sql server
- substring
meta:
  _edit_last: '1'
  _thumbnail_id: '403'
  _wp_old_slug: repurposing-sql-server-parsename-for-splitting-ipv4-addresses
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p>[caption id="attachment_403" align="alignright" width="128" caption="Geo IP"]<img class="size-full wp-image-403" title="Geo IP" src="{{ site.baseurl }}/assets/geoip.png" alt="Geo IP" width="128" height="128" />[/caption]</p>
<p style="text-align: justify;">I stumbled across a very nice repurposing of the PARSENAME function in SQL Server recently while playing around with some GeoIP Data. In SQL Server, the PARSENAME function is used for working with fully qualified server objects. e.g. a table on a linked server ('LinkedServerName . Databasename . Ownername . TableName'). But PARSENAME can be used to easily split up any 4 token, dot delimited string into its constituent parts.</p>
<p><!--more--><br /></p>
<p style="text-align: justify;">It's intended use, is for dealing with the string representations of server objects. e.g.</p>
<p>
<strong>SQL</strong></p>

```sql
DECLARE @MyObject VARCHAR(100)
SET @MyObject = 'MyServer.MyDB.MyOwner.MyTable'

SELECT PARSENAME(@MyObject, 4) AS [ServerName],
	PARSENAME(@MyObject, 3) AS [DatabaseName],
	PARSENAME(@MyObject, 2) AS [OwnerName],
	PARSENAME(@MyObject, 1) As [TableName]
```

<p>
<strong>Results</strong></p>

```
ServerName   DatabaseName     OwnerName    TableName
----------- ---------------- ------------ ----------
MyServer     MyDB             MyOwner      MyTable

(1 row(s) affected)
``` 

<p style="text-align: justify;">A nice side effect of it's functionality, is that being useful for splitting any 4 part, dot delimited string into it's constiuent parts... e.g. an IPv4 Address; essentially saving you from performing charindex/substring gymnastics in order to split the string.</p>
<p>
<strong>SQL</strong></p>

```sql
DECLARE @MyIP VARCHAR(15)
SET @MyIP = '1.12.34.45'

SELECT
	PARSENAME(@MyIP , 4) AS [octet1],
	PARSENAME(@MyIP , 3) AS [octet2],
	PARSENAME(@MyIP , 2) AS [octet3],
	PARSENAME(@MyIP , 1) As [octet4]
```
<p>
<strong>Results</strong></p>

```
octet1    octet2    octet3   octet4
--------- --------- ------------------
1         12        34       45

(1 row(s) affected)
```
<p style="text-align: justify;">If you're using the <a href="http://www.maxmind.com/app/geoip_country">MaxMind GeoIP Lite Database</a>, you can very quickly access each IP range by converting the IP address to it's equivalent Integer value and doing an indexed search for that value.</p>

```
StringIP = oc4.oc3.oc2.oc1
IntIP = (oc4 * (256^3)) + (oc3 * (256^2))
       + (oc2 * (256^1)) + (oc1 * (256^0))
```

<p>Here's a sql function that'll do just that.</p>

```sql
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE FUNCTION [dbo].[IP_2_INT](@ip varchar(15))
RETURNS INT
WITH SCHEMABINDING
AS
BEGIN
    DECLARE @ipInt INT, @oa TINYINT, @ob TINYINT, @oc TINYINT, @od TINYINT;

    SELECT @od = CAST(PARSENAME(@ip, 4) AS TINYINT),
     @oc = CAST(PARSENAME(@ip, 3) AS TINYINT),
     @ob = CAST(PARSENAME(@ip, 2) AS TINYINT),
     @oa = CAST(PARSENAME(@ip, 1) AS TINYINT)

    SET @ipInt = (@od*256*256*256) + (@oc*256*256) + (@ob*256) + @oa;
    RETURN @ipInt;
END
```

<p></p>
<p style="text-align: justify;">There is a small downside to this. Unfortunately the PARSENAME function is <a href="http://msdn.microsoft.com/en-us/library/ms178091.aspx">non-deterministic</a> meaning it could potentially return different results when called across different servers.</p>

```sql
SELECT OBJECTPROPERTYEX(OBJECT_ID('dbo.IP_2_INT'), 'IsDeterministic')
``` 

<p></p>
<p style="text-align: justify;">This was incorrectly documented in that past and so if it#s an issue for you then there is a work around using good old charindex &amp; substrings available on the microsoft connect bug report page for it.<br />
<a href="http://connect.microsoft.com/SQLServer/feedback/details/488058/parsename-incorrectly-documented-as-deterministic#details">PARSENAME incorrectly documented as deterministic</a></p>
<p>
<em>~Eoin Campbell</em></p>
