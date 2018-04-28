---
title: "Powershell Notes"
date: 2018-04-27T20:28:51-07:00
tags:
    - powershell
draft: false
---

## Where-Object (where)

Filters objects based on specified search criteria.

```powershell
Get-Service | where Status -eq 'running'
```

is equivalent to

```powershell
Get-Service | where {$_.Status -eq 'running'}
```

Note: `$_` is the current object in the pipeline.

[Comparison operators](https://technet.microsoft.com/en-us/library/hh847759.aspx)
I find these are the most commonly used operators:

* -ne (not equal to)
* -eq (equal to)
* -like (a wildcard comparison)

If comparison operator is not defined, the `where` clause returns all objects that have the specified property defined.

## Select-Object (select)

```powershell
Get-Command | select Name, Version -First 3
```

## Select-String (sls)

```powershell
sls *.yml -Pattern 'hexo' -CaseSensitive -Context 1
```

Performs case sensitive searches for lines that contain pattern `hexo` in all `*.yml` files. For each line that matches the pattern also print one line before and after.

## ForEach-Object (foreach or %)

```powershell
gci *.yml | %{$_.Name}
```

## Get-Command (gm)

Lists property of objects in the pipeline

```powershell
gci *.yml | gm
```

The output is

```
   TypeName: System.IO.FileInfo

Name                      MemberType     Definition
----                      ----------     ----------
LinkType                  CodeProperty   System.String LinkType{get=GetLinkType;}
Mode                      CodeProperty   System.String Mode{get=Mode;}
...
```

## Send output to clipboard

Use `clip` utility

```powershell
Get-EventLog application -Newest 1 | clip
```
