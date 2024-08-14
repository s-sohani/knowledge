**System-Wide Limits:**

Edit `/etc/security/limits.conf`:
`* soft nofile 100000 * hard nofile 100000`

Edit `/etc/sysctl.conf`:
`fs.file-max = 2097152`

Apply the changes:
`sudo sysctl -p`