---
layout: post
title: "The most 'find' you will need"
date: 2025-03-10 00:00:00 +0100
categories: [system, hacking, linux]
tags: [search, find, bash]
---

![path](pics/path-03-10.png)

# Basic Syntax
```
find [path] [options] [expression]

- [path] → Where to search (e.g., /home, ., /var/log)
- [options] → Conditions like -name, -type, -size, etc.
- [expression] → Additional actions like -exec, -delete, etc.
```
### Basic xamples of finding files

Finding Files by Name inside /home/user
```bash
find /home/user -name "file.txt"
```
Let's ignore case sensitivity
```bash
find /home/user -iname "file.txt"
```
Find all *.sh* files by name
```bash
find / -type f  -name "*.sh"
```
Limit search depth to 2 directories
```bash
find /home/user -maxdepth 2 -type f -name "*.txt"
```

### Finding Files by type

Find only directories
```bash
find /path/to/search -type d
```
Find only regular files
```bash
find /path/to/search -type f
```
Find symbolic links
```bash
find /path/to/search -type l
```

### Finding Files by Size

Find files larger than 100MB
```bash
find / -type f -size +100M
```
Find files smaller than 1KB
```bash
find / -type f -size -1k
```
Find files of exactly 50MB
```bash
find / -type f -size 50M
```

### Finding Files by Modification Time

Find files modified in the last 2 days
```bash
find /home/user -type f -mtime -2
```
Find files modified more than 30 days ago
```bash
find /home/user -type f -mtime +30
```
Find files accessed in the last 10 days
```bash
find /home/user -type f -atime -10
```
Find files changed in the last 1 hour
```bash
find /home/user -type f -cmin -60
```
Find files modified after a specific date
```bash
find / -type f -newermt '6/30/2020 0:00:00'
```
*(all dates/times after 6/30/2020 0:00:00 will be considered a condition to look for)*

Find files based on date modified
find [directory path] -type f -newermt [start date range] ! -newermt [end date range]
```bash
find / -type f -newermt 2013-09-12 ! -newermt 2013-09-14
```
*(all dates before 2013-09-12 will be excluded; all dates after 2013-09-14 will be excluded, therefore this only leaves 2013-09-13 as the date to look for.)*

Find files based on date accessed
find [directory path] -type f -newerat [start date range] ! -newerat [end date range]
```bash
find / -type f -newerat 2017-09-12 ! -newerat 2017-09-14
```
(all dates before 2017-09-12 will be excluded; all dates after 2017-09-14 will be excluded, therefore this only leaves 2017-09-13 as the date to look for.)


### Finding Files by Permissions

Files with the SUID bit allow users to execute the file with the permissions of the file owner (usually root).
```bash
find / -type f -perm -4000 2>/dev/null
```
or
```bash
find / -type f -perm -u=s 2>/dev/null
```
Find SUID files owned by root
```bash
find / -type f -perm -4000 -user root 2>/dev/null
```
Files that anyone can write to (777 permission) can be exploited.
```bash
find / -type f -perm -o=w 2>/dev/null
```
Find files with exact permission (e.g., 755)
```bash
find / -type f -perm 755
```
Find files owned by a specific user
```bash
find / -type f -user username
```
Find files owned by a specific group
```bash
find / -type f -group groupname
```
```
/u=s → Finds files where the user (owner) has the SUID bit set.
/g=s → Finds files where the group has the SGID bit set.
/u=s,g=s → Finds files with either SUID or SGID.
```

### Not sure when you will need these but Finding Empty Files and Directories

Find empty files
```bash
find /home/user -type f -empty
```
Find empty directories
```bash
find /home/user -type d -empty
```
