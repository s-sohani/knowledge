To avoid being prompted for a password for each database when performing a PostgreSQL dump, you can use a few methods:

1. **PGPASSWORD Environment Variable**: Set the `PGPASSWORD` environment variable in your shell session before running the dump command. For example:   
    `export PGPASSWORD='your_password' pg_dumpall -U your_username > alldatabases.sql`
    
2. **.pgpass File**: You can create a `.pgpass` file in your home directory, which stores passwords for different database connections. The file should have the following format:
    `hostname:port:database:username:password`
    
    For example, if you want to set the password for all databases for a specific user:  
    `*:*:*:your_username:your_password`
    
    Make sure to set the file permissions to be readable only by you:
    `chmod 600 ~/.pgpass`
    
3. **Using a Connection String**: You can also provide the password directly in the connection string when using `pg_dumpall`, although this is not recommended due to security reasons:
    `pg_dumpall -U your_username -h localhost -W 'your_password' > alldatabases.sql`


