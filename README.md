# snowflake-sample-dbt

Minimal dbt project (dbt 1.8+) over Snowflake’s sample database `SNOWFLAKE_SAMPLE_DATA`, schema `TPCH`. One staging view per source table: `customer`, `orders`, `lineitem`, `part`, `supplier`, `partsupp`, `nation`, and `region`.

Primary-key tests are `not_null` and `unique`. Composite keys (`lineitem`, `partsupp`) use a small generic `unique_combination_of_columns` test in this repo, so there is no package install.

Views are created in the database and schema from your profile. The sample database is read-only source data.

## Point a profile at a query role

Use a Snowflake **query role**. It should be able to read the sample tables and create views in a database you own. It should not be a role that can modify `SNOWFLAKE_SAMPLE_DATA`.

The role needs:

- `USAGE` on the warehouse
- `USAGE` on database `SNOWFLAKE_SAMPLE_DATA` and schema `TPCH`, plus `SELECT` on those tables
- `USAGE` on the target database and `CREATE VIEW` on the target schema

Copy the example and keep secrets in the environment. `profiles.yml` is gitignored.

```bash
cp profiles.yml.example profiles.yml
export SNOWFLAKE_ACCOUNT=your_account
export SNOWFLAKE_USER=your_user
export SNOWFLAKE_ROLE=your_query_role
export SNOWFLAKE_WAREHOUSE=your_warehouse
export SNOWFLAKE_DATABASE=your_database
export SNOWFLAKE_SCHEMA=your_schema
export SNOWFLAKE_PASSWORD=your_password
```

`SNOWFLAKE_ROLE` is the query role. To use key-pair auth instead of a password, comment out `password` in `profiles.yml` and set `private_key_path`.

dbt looks for `profiles.yml` in `~/.dbt/` by default. Either copy the file there or point dbt at this directory:

```bash
export DBT_PROFILES_DIR="$PWD"
```

The profile name is `snowflake_sample_dbt`, matching `dbt_project.yml`.

## Build

Install dbt 1.8 or newer with the Snowflake adapter (`dbt-snowflake`), then from this directory:

```bash
dbt build
```

`dbt build` creates the eight staging views and runs the primary-key tests.
