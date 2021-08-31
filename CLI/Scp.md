### Copy file from remote with `scp`
``` bash
scp user@host:/path/to/destination /path/to/file
# specify private key
scp -i .ssh/deploy Export.zip user@host:\home\deploy Export.zip
```
### Copy mutiple files or directory
```bash
# copying multiple files using scp command
scp file1 file2 user@host:/path/to/directory
# Copying an entire directory with scp command
scp -r /path/to/directory user@host:/path/to/directory
```