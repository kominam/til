### Usage
```bash
find <path> <conditions> <actions>
```
### Access time conditions
```bash
# Last accessed between now and 24 hours ago
-atime 0
# Accessed more than 24 hours ago
-atime +0
# Accessed between 24 and 48 hours ago
-atime 1
# Accessed more than 48 hours ago
-atime +1
# Accessed less than 24 hours ago (same a 0)
-atime -1
# File status changed within the last 6 hours and 30 minutes
-ctime -6h30m
# Last modified more than 1 week ago
-mtime +1w
```
### Condition flow
```bash
\! -name "*.c"
\( x -or y \)
```
### Actions
```bash
-exec rm {} \;
-print
-delete
```
### Conditions
```bash
-name "*.c"
```

```bash
-type f            # File
-type d            # Directory
-type l            # Symlink
```

```bash
-depth 2           # At least 3 levels deep
-regex PATTERN
```

```bash
-newer   file.txt
-newerm  file.txt        # modified newer than file.txt
-newerX  file.txt        # [c]hange, [m]odified, [B]create
-newerXt "1 hour ago"    # [t]imestamp
```
### Examples
```bash
find . -name '*.jpg'
find . -name '*.jpg' -exec rm {} \;
```

```bash
find . -newerBt "24 hours ago"
```

```bash
find . -type f -mtime +29 # find files modified more than 30 days ago
```