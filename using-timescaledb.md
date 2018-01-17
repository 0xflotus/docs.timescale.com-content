# Using TimescaleDB

TimescaleDB focuses on _simplicity_ for our users and how they can operate and manage their database, infrastructure, and applications, especially at scale.

First and foremost, developed TimescaleDB as an extension to PostgreSQL,
rather than building a time-series database from scratch. We also chose not to introduce
our own custom query language. Instead, TimescaleDB fully embraces SQL.

TimescaleDB supports all SQL operations and queries one would expect out of PostgreSQL.
This includes how tables are created, altered and deleted, how schemas are built and indexed,
and how data is inserted and queried. Additionally, TimescaleDB adds necessary and useful
functions for operational ease-of-use and analytical flexibility. In general, if you are
familiar with SQL, TimescaleDB will be familiar to you.

The most important design aspect for providing users with a simple interface to
the database is the TimescaleDB hypertable, discussed earlier within
[Architecture & Concepts][architecture].

Essentially, hypertables abstract away the complexity of TimescaleDB's automatic
partitioning, so users don't have to worry about managing any of the underlying
chunks individually. Instead, users can focus on developing and interacting with their data as
they would with regular tables within a PostgreSQL database. For advanced users, TimescaleDB is
transparent about the presence of chunks and allows several ways to access them directly.
This section covers all of the operations, and more, for using TimescaleDB. Again, for
those coming from the world of SQL and PostgreSQL, these docs will be refreshingly simple.

[architecture]: /introduction/architecture
[creating-hypertables]: /using-timescaledb/hypertables
