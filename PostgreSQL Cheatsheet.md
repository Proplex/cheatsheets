# PostgreSQL Cheatsheet
#work #cheatsheet #postgresql


## Enable citext for CF Deployments
Grab the Postgres information from wherever you have it stored (IP, User, Pass) and run `psql postgres://<user>:<password>@<ip>`

Then, for each of the databases you want to enable citext on (for CF that’s  `diegodb` ,  `uaadb` , and `ccdb`), run:

```
\c <db name>
create extension citext;
```

## List all Postgres Databases
`\l`

wow. isn’t that nice.