# Reset password of user postgres on local for Mac

1. Find out which postgres is running and find out where is the installed folder
`ps aux | grep postgres`
2. Login using `su`
`sudo su - postgres`
3. Stop PostgreSql service
`/Library/PostgreSQL/16/bin/pg_ctl stop -D /Library/PostgreSQL/16/data`
4. Enter single user mode
`/Library/PostgreSQL/16/bin/postgres --single -D /Library/PostgreSQL/16/data postgres`
5. Prompt will be changed to `backend>`
6. Update the password
`ALTER USER postgres WITH PASSWORD 'newpassword';`
7. `Ctrl+D` to exit
8. Start the service
`/Library/PostgreSQL/16/bin/pg_ctl start -D /Library/PostgreSQL/16/data`
9. Use `exit` to exit