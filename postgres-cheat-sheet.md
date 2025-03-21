# Postgres Cheat Sheet

<!-- TOC -->

- [Postgres Cheat Sheet](#postgres-cheat-sheet)
	- [Setup Database Server](#setup-database-server)
		- [Start up postgre server](#start-up-postgre-server)
		- [Custom config](#custom-config)
	- [Connect to database server](#connect-to-database-server)
		- [Prerequiste](#prerequiste)
		- [Connect to the server using psql](#connect-to-the-server-using-psql)
		- [Connect and execute SQL file](#connect-and-execute-sql-file)
	- [Configuration](#configuration)
		- [Default Configuration file (~/.psqlrc)](#default-configuration-file-psqlrc)
		- [toggle expanded output (default off)](#toggle-expanded-output-default-off)
	- [Database](#database)
		- [list databases](#list-databases)
		- [switch databases to connect](#switch-databases-to-connect)
	- [Table, View, and Index](#table-view-and-index)
		- [list tables](#list-tables)
		- [describe table, view, sequence, or index](#describe-table-view-sequence-or-index)
		- [list access privileges of table, view, and sequence](#list-access-privileges-of-table-view-and-sequence)
		- [show view definition](#show-view-definition)
	- [Users](#users)
		- [display current user (role)](#display-current-user-role)
		- [list users (roles)](#list-users-roles)
		- [grant privileges to user (role)](#grant-privileges-to-user-role)
		- [switch user](#switch-user)
	- [Help](#help)
		- [help: psql command options](#help-psql-command-options)
		- [help on psql backslach commands](#help-on-psql-backslach-commands)
	- [Useful resources](#useful-resources)

<!-- /TOC -->

## Setup Database Server

### Start up postgre server

> docker-compose.yml
```yaml
version: '3'

services:
  db:
    image: postgres
    container_name: postgres
    ports:
      - 5432:5432
    volumes:
      - db-store:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=pass
volumes:
  db-store:
```

Startup the server like this:

```bash
docker-compoase up -d
# finch compse up -d
```

Finally connnect to the server

```bash
# use local psql
psql -h localhost -U postgres

# you can exec container and connect
docker exec -it postgres /bin/sh
$ psql -h localhost -U hoge -d hogedb
```

### Custom config
```bash
#  get the default config
docker run -i --rm postgres cat /usr/share/postgresql/postgresql.conf.sample > my-postgres.conf
# finch run -i --rm postgres cat /usr/share/postgresql/postgresql.conf.sample > my-postgres.conf

# customize the config

# run postgres with custom config
docker run -d --name some-postgres -v "$PWD/my-postgres.conf":/etc/postgresql/postgresql.conf -e POSTGRES_PASSWORD=mysecretpassword postgres -c 'config_file=/etc/postgresql/postgresql.conf'

```

ref: https://hub.docker.com/_/postgres

## Connect to database server

### Prerequiste

You need `psql` command to connect to database server.

You can install it locally

```bash
# macos
brew install libpq

echo 'export PATH="/usr/local/opt/libpq/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

psql --version
```

Or you can leverage a container that has psql inside like this:

```bash
docker exec -it <container>  bash
finch exec -it <container>  bash
```

See also [Running Postgres Client in Kubernetes](https://gist.github.com/yokawasa/5358e79636d480274b8731a050ff5a98)

### Connect to the server using psql

```
psql -U postgres
psql -h localhost -p 5432 -U postgres -W

psql \
   --host=<DB instance endpoint> \
   --port=<port> \
   --username=<master username> \
   --password \
   --dbname=<database name>
```

### Connect and execute SQL file

```bash
# pass the sql via pipe
cat sql/hogedb.sql|  psql -h localhost -U postgres

# HERE document pattern
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
	CREATE USER docker;
	CREATE DATABASE docker;
	GRANT ALL PRIVILEGES ON DATABASE docker TO docker;
EOSQL
```

## Configuration

### Default Configuration file (~/.psqlrc)

If you write the above settings in the `~/.psqlrc` file, they will be read as default settings.

```
\x auto
\set COMP_KEYWORD_CASE upper

\set ECHO none
\set PROMPT1 '%n@%/%R%# %x '
\set ON_ERROR_STOP on
\set ON_ERROR_ROLLBACK interactive

\pset null '¤'
\pset linestyle 'unicode'

\pset unicode_border_linestyle single
\pset unicode_column_linestyle single
\pset unicode_header_linestyle double

set intervalstyle to 'postgres_verbose';

\setenv LESS '-iMFXSx4R'
\set ECHO all
```

ref: https://github.com/andrewkslv/postgresql-cheat-sheet

### toggle expanded output (default off)

By turning on the expanded display mode, the results will be displayed vertically.

```bash
\x [on|off|auto]

# always on
\x on
# always off
\x off
# The results will be displayed vertically when the result of select is too long
\x auto
```

## Database

### list databases

```
\l
```

### switch databases to connect

```
\c <database>
\connect <database>
```

## Table, View, and Index

### list tables

list tables (including view & sequence)

```
\d
```

list tables

```
\dt
```

### describe table, view, sequence, or index

```
\d <table>
```

### list access privileges of table, view, and sequence

`\z` and `\dp` are the same

```
\z <table>
\dp <table>
```

### show view definition

```sql
select definition from pg_views where viewname = '<VIEW_NAME>';
```

## Users

### display current user (role)

```sql
select current_user;
```

### list users (roles)

```
\du
```

Or you can get list of user info with sql statement like this:

```sql
select * from pg_user;

usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  | valuntil | useconfig 
----------+----------+-------------+----------+---------+--------------+----------+----------+-----------
 postgres |       10 | t           | t        | t       | t            | ******** |          | 
(1 row)
```

### grant privileges to user (role)

```sql
grant select, insert, update, delete on <TABLE_NAME> to <USER_NAME>;
```

### switch user

```
\c - <USER_NAME>
\connect - <USER_NAME>
```




## Help

### help: psql command options

<details><summary>output: psql --help</summary>
<p>

```
bash-5.1# psql --help
psql is the PostgreSQL interactive terminal.

Usage:
  psql [OPTION]... [DBNAME [USERNAME]]

General options:
  -c, --command=COMMAND    run only single command (SQL or internal) and exit
  -d, --dbname=DBNAME      database name to connect to (default: "root")
  -f, --file=FILENAME      execute commands from file, then exit
  -l, --list               list available databases, then exit
  -v, --set=, --variable=NAME=VALUE
                           set psql variable NAME to VALUE
                           (e.g., -v ON_ERROR_STOP=1)
  -V, --version            output version information, then exit
  -X, --no-psqlrc          do not read startup file (~/.psqlrc)
  -1 ("one"), --single-transaction
                           execute as a single transaction (if non-interactive)
  -?, --help[=options]     show this help, then exit
      --help=commands      list backslash commands, then exit
      --help=variables     list special variables, then exit

Input and output options:
  -a, --echo-all           echo all input from script
  -b, --echo-errors        echo failed commands
  -e, --echo-queries       echo commands sent to server
  -E, --echo-hidden        display queries that internal commands generate
  -L, --log-file=FILENAME  send session log to file
  -n, --no-readline        disable enhanced command line editing (readline)
  -o, --output=FILENAME    send query results to file (or |pipe)
  -q, --quiet              run quietly (no messages, only query output)
  -s, --single-step        single-step mode (confirm each query)
  -S, --single-line        single-line mode (end of line terminates SQL command)

Output format options:
  -A, --no-align           unaligned table output mode
  -F, --field-separator=STRING
                           field separator for unaligned output (default: "|")
  -H, --html               HTML table output mode
  -P, --pset=VAR[=ARG]     set printing option VAR to ARG (see \pset command)
  -R, --record-separator=STRING
                           record separator for unaligned output (default: newline)
  -t, --tuples-only        print rows only
  -T, --table-attr=TEXT    set HTML table tag attributes (e.g., width, border)
  -x, --expanded           turn on expanded table output
  -z, --field-separator-zero
                           set field separator for unaligned output to zero byte
  -0, --record-separator-zero
                           set record separator for unaligned output to zero byte

Connection options:
  -h, --host=HOSTNAME      database server host or socket directory (default: "local socket")
  -p, --port=PORT          database server port (default: "5432")
  -U, --username=USERNAME  database user name (default: "root")
  -w, --no-password        never prompt for password
  -W, --password           force password prompt (should happen automatically)

For more information, type "\?" (for internal commands) or "\help" (for SQL
commands) from within psql, or consult the psql section in the PostgreSQL
documentation.
```

</p></details>

### help on psql backslach commands
<details><summary>output: \?</summary>
<p>

```
General
  \copyright             show PostgreSQL usage and distribution terms
  \crosstabview [COLUMNS] execute query and display result in crosstab
  \errverbose            show most recent error message at maximum verbosity
  \g [(OPTIONS)] [FILE]  execute query (and send result to file or |pipe);
                         \g with no arguments is equivalent to a semicolon
  \gdesc                 describe result of query, without executing it
  \gexec                 execute query, then execute each value in its result
  \gset [PREFIX]         execute query and store result in psql variables
  \gx [(OPTIONS)] [FILE] as \g, but forces expanded output mode
  \q                     quit psql
  \watch [SEC]           execute query every SEC seconds

Help
  \? [commands]          show help on backslash commands
  \? options             show help on psql command-line options
  \? variables           show help on special variables
  \h [NAME]              help on syntax of SQL commands, * for all commands

Query Buffer
  \e [FILE] [LINE]       edit the query buffer (or file) with external editor
  \ef [FUNCNAME [LINE]]  edit function definition with external editor
  \ev [VIEWNAME [LINE]]  edit view definition with external editor
  \p                     show the contents of the query buffer
  \r                     reset (clear) the query buffer
  \s [FILE]              display history or save it to file
  \w FILE                write query buffer to file

Input/Output
  \copy ...              perform SQL COPY with data stream to the client host
  \echo [-n] [STRING]    write string to standard output (-n for no newline)
  \i FILE                execute commands from file
  \ir FILE               as \i, but relative to location of current script
  \o [FILE]              send all query results to file or |pipe
  \qecho [-n] [STRING]   write string to \o output stream (-n for no newline)
  \warn [-n] [STRING]    write string to standard error (-n for no newline)

Conditional
  \if EXPR               begin conditional block
  \elif EXPR             alternative within current conditional block
  \else                  final alternative within current conditional block
  \endif                 end conditional block

Informational
  (options: S = show system objects, + = additional detail)
  \d[S+]                 list tables, views, and sequences
  \d[S+]  NAME           describe table, view, sequence, or index
  \da[S]  [PATTERN]      list aggregates
  \dA[+]  [PATTERN]      list access methods
  \dAc[+] [AMPTRN [TYPEPTRN]]  list operator classes
  \dAf[+] [AMPTRN [TYPEPTRN]]  list operator families
  \dAo[+] [AMPTRN [OPFPTRN]]   list operators of operator families
  \dAp[+] [AMPTRN [OPFPTRN]]   list support functions of operator families
  \db[+]  [PATTERN]      list tablespaces
  \dc[S+] [PATTERN]      list conversions
  \dconfig[+] [PATTERN]  list configuration parameters
  \dC[+]  [PATTERN]      list casts
  \dd[S]  [PATTERN]      show object descriptions not displayed elsewhere
  \dD[S+] [PATTERN]      list domains
  \ddp    [PATTERN]      list default privileges
  \dE[S+] [PATTERN]      list foreign tables
  \des[+] [PATTERN]      list foreign servers
  \det[+] [PATTERN]      list foreign tables
  \deu[+] [PATTERN]      list user mappings
  \dew[+] [PATTERN]      list foreign-data wrappers
  \df[anptw][S+] [FUNCPTRN [TYPEPTRN ...]]
                         list [only agg/normal/procedure/trigger/window] functions
  \dF[+]  [PATTERN]      list text search configurations
  \dFd[+] [PATTERN]      list text search dictionaries
  \dFp[+] [PATTERN]      list text search parsers
  \dFt[+] [PATTERN]      list text search templates
  \dg[S+] [PATTERN]      list roles
  \di[S+] [PATTERN]      list indexes
  \dl[+]                 list large objects, same as \lo_list
  \dL[S+] [PATTERN]      list procedural languages
  \dm[S+] [PATTERN]      list materialized views
  \dn[S+] [PATTERN]      list schemas
  \do[S+] [OPPTRN [TYPEPTRN [TYPEPTRN]]]
                         list operators
  \dO[S+] [PATTERN]      list collations
  \dp     [PATTERN]      list table, view, and sequence access privileges
  \dP[itn+] [PATTERN]    list [only index/table] partitioned relations [n=nested]
  \drds [ROLEPTRN [DBPTRN]] list per-database role settings
  \dRp[+] [PATTERN]      list replication publications
  \dRs[+] [PATTERN]      list replication subscriptions
  \ds[S+] [PATTERN]      list sequences
  \dt[S+] [PATTERN]      list tables
  \dT[S+] [PATTERN]      list data types
  \du[S+] [PATTERN]      list roles
  \dv[S+] [PATTERN]      list views
  \dx[+]  [PATTERN]      list extensions
  \dX     [PATTERN]      list extended statistics
  \dy[+]  [PATTERN]      list event triggers
  \l[+]   [PATTERN]      list databases
  \sf[+]  FUNCNAME       show a function's definition
  \sv[+]  VIEWNAME       show a view's definition
  \z      [PATTERN]      same as \dp

Large Objects
  \lo_export LOBOID FILE write large object to file
  \lo_import FILE [COMMENT]
                         read large object from file
  \lo_list[+]            list large objects
  \lo_unlink LOBOID      delete a large object

Formatting
  \a                     toggle between unaligned and aligned output mode
  \C [STRING]            set table title, or unset if none
  \f [STRING]            show or set field separator for unaligned query output
  \H                     toggle HTML output mode (currently off)
  \pset [NAME [VALUE]]   set table output option
                         (NAME := {border|columns|expanded|fieldsep|fieldsep_zero|
                         footer|format|linestyle|null|numericlocale|pager|
                         pager_min_lines|recordsep|recordsep_zero|tableattr|title|
                         tuples_only|unicode_border_linestyle|
                         unicode_column_linestyle|unicode_header_linestyle})
  \t [on|off]            show only rows (currently off)
  \T [STRING]            set HTML <table> tag attributes, or unset if none
  \x [on|off|auto]       toggle expanded output (currently off)
          
Connection
  \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                         connect to new database (currently "postgres")
  \conninfo              display information about current connection
  \encoding [ENCODING]   show or set client encoding
  \password [USERNAME]   securely change the password for a user
          
Operating System
  \cd [DIR]              change the current working directory
```

</p></details>

## Useful resources

- https://postgrescheatsheet.com/
- https://qiita.com/kiyodori/items/b9b34c7abe7e45f65c46