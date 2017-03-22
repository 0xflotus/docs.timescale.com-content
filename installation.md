# Installation

TimescaleDB is packaged as a PostgreSQL extension and set of scripts.

There are two ways to install TimescaleDB: (1) Docker and (2) Postgres.

## Installation (from source)

_NOTE: Currently, upgrading to new versions of TimescaleDB requires a fresh install._

Clone the repository from our [Github site][Github] and navigate to the repo base directory.

### Option 1. Docker (recommended)

**Prerequisites**

- [Postgres client][Postgres-client] (psql)

- [Docker][]

**Build and run in Docker**

```bash
# To build a Docker image
make -f docker.mk build-image

# To run a container
make -f docker.mk run
```

### Option 2. Local Postgres

**Prerequisites**

- A standard **PostgreSQL 9.6** installation with development environment (header files) (e.g., [Postgres.app for MacOS][Postgres-app])

**Build and install with local PostgreSQL**

```bash
# To build the extension
make

# To install
make install
```

**Update `postgresql.conf`**

Also, you will need to edit your `postgresql.conf` file to include
necessary libraries, and then restart PostgreSQL:
```bash
# Modify postgresql.conf to add required libraries. For example,
shared_preload_libraries = 'dblink,timescaledb'

# Then, restart PostgreSQL
```

## Setting up your initial database
You have two options for setting up your initial database:
1. *Empty Database* - To set up a new, empty database, please follow the instructions below.

2. *Database with pre-loaded sample data* - To help you quickly get started, we have also created some sample datasets.
See [Sample Datasets][datasets] for further instructions. (Includes installing our extension.)

[datasets]: /other-sample-datasets

### Setting up an empty database

When creating a new database, it is necessary to install the extension and then run an initialization function.  Here we will create a new database named "tutorial".

```bash
# Connect to Postgres as a superuser named 'postgres'
psql -U postgres -h localhost
```

```sql
-- Install the extension
CREATE database tutorial;

\c tutorial

CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;

-- Run initialization function
select setup_timescaledb();
```

For convenience, this can also be done in one step by running a script from
the command-line:
```bash
DB_NAME=tutorial ./scripts/setup-db.sh
```

You should now have a brand new time-series database running in Postgres.

```bash
# To access your new database
psql -U postgres -h localhost -d tutorial
```

Next let's load some data.

## Working with time-series data

One of the core ideas of our time-series database are time-series optimized data tables, called **hypertables**.

### Creating a (hyper)table
To create a hypertable, you start with a regular SQL table, and then convert
it into a hypertable via the function
`create_hypertable()`([API definition](/api-docs)).

The following example creates a hypertable for tracking
temperature and humidity across a collection of devices over time.

```sql
-- We start by creating a regular SQL table
CREATE TABLE conditions (
  time        TIMESTAMP WITH TIME ZONE NOT NULL,
  device_id   TEXT                     NOT NULL,
  temperature DOUBLE PRECISION         NULL,
  humidity    DOUBLE PRECISION         NULL
);
```

Next, transform it into a hypertable using the provided function
`create_hypertable()`:

```sql
-- This creates a hypertable that is partitioned by time
--   using the values in the `time` column.
SELECT create_hypertable('conditions', 'time');

-- OR you can additionally partition the data on another dimension
--   (what we call 'space') such as `device_id`.
-- For example, to partition `device_id` into 2 partitions:
SELECT create_hypertable('conditions', 'time', 'device_id', 2);
```

### Inserting and querying
Inserting data into the hypertable is done via normal SQL `INSERT` commands,
e.g. using millisecond timestamps:
```sql
INSERT INTO conditions(time,device_id,temperature,humidity)
VALUES(NOW(), 'office', 70.0, 50.0);
```

Similarly, querying data is done via normal SQL `SELECT` commands.
SQL `UPDATE` and `DELETE` commands also work as expected.

### Indexing data

Data is indexed using normal SQL `CREATE INDEX` commands. For instance,
```sql
CREATE INDEX ON conditions (device_id, time DESC);
```
This can be done before or after converting the table to a hypertable.

**Indexing suggestions:**

Our experience has shown that different types of indexes are most-useful for
time-series data, depending on your data.

For indexing columns with discrete (limited-cardinality) values
(e.g., where you are most likely to use an "equals" or "not equals" comparator)
we suggest using an index like this (using our hypertable `conditions` for the example):
```sql
CREATE INDEX ON conditions (device_id, time DESC);
```
For all other types of columns, i.e., columns with continuous values (e.g., where you are most likely to use a
"less than" or "greater than" comparator) the index should be in the form:
```sql
CREATE INDEX ON conditions (time DESC, temperature);
```
Having a `time DESC` column specification in the index allows for efficient
queries by column-value and time. For example, the index defined above would
optimize the following query:
```sql
SELECT * FROM conditions WHERE device_id = 'dev_1' ORDER BY time DESC LIMIT 10
```

For sparse data where a column is often NULL, we suggest adding a
`WHERE column IS NOT NULL` clause to the index (unless you are often
searching for missing data). For example,

```sql
CREATE INDEX ON conditions (time DESC, humidity) WHERE humidity IS NOT NULL;
```
this creates a more compact, and thus efficient, index.

[Github]: https://www.github.com/timescaledb/timescaledb

<!-- Prerequisites -->
[Postgres-client]: https://wiki.postgresql.org/wiki/Detailed_installation_guides
[Docker]: https://docs.docker.com/engine/installation/
[Postgres-app]:  https://postgresapp.com/
