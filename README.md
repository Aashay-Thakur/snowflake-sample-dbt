# snowflake-sample-dbt

Minimal dbt project (dbt 1.8+) over Snowflake’s sample database `SNOWFLAKE_SAMPLE_DATA`, schema `TPCH_SF10`. One staging view per source table: `customer`, `orders`, `lineitem`, `part`, `supplier`, `partsupp`, `nation`, and `region`.

Primary-key tests are `not_null` and `unique`. Composite keys (`lineitem`, `partsupp`) use a small generic `unique_combination_of_columns` test in this repo, so there is no package install.

Views are created in the database and schema from your profile. The sample database is read-only source data.

## Point a profile at a query role

Use a Snowflake **query role**. It should be able to read the sample tables and create views in a database you own. It should not be a role that can modify `SNOWFLAKE_SAMPLE_DATA`.

The role needs:

- `USAGE` on the warehouse
- `USAGE` on database `SNOWFLAKE_SAMPLE_DATA` and schema `TPCH_SF10`, plus `SELECT` on those tables
- `USAGE` on the target database and `CREATE VIEW` on the target schema

Copy the example and keep secrets in the environment. `profiles.yml` is gitignored. Run these commands from this directory (the folder that contains `dbt_project.yml`).

bash:

```bash
cp profiles.yml.example profiles.yml
export SNOWFLAKE_ACCOUNT=your_account
export SNOWFLAKE_USER=your_user
export SNOWFLAKE_ROLE=your_query_role
export SNOWFLAKE_WAREHOUSE=your_warehouse
export SNOWFLAKE_DATABASE=your_database
export SNOWFLAKE_SCHEMA=your_schema
export SNOWFLAKE_PASSWORD=your_password
export DBT_PROFILES_DIR="$PWD"
```

PowerShell:

```powershell
Copy-Item profiles.yml.example profiles.yml
$env:SNOWFLAKE_ACCOUNT = "your_account"
$env:SNOWFLAKE_USER = "your_user"
$env:SNOWFLAKE_ROLE = "your_query_role"
$env:SNOWFLAKE_WAREHOUSE = "your_warehouse"
$env:SNOWFLAKE_DATABASE = "your_database"
$env:SNOWFLAKE_SCHEMA = "your_schema"
$env:SNOWFLAKE_PASSWORD = "your_password"
$env:DBT_PROFILES_DIR = (Get-Location).Path
```

`SNOWFLAKE_ROLE` is the query role. To use key-pair auth instead of a password, comment out `password` in `profiles.yml` and set `private_key_path` to `{{ env_var('SNOWFLAKE_PRIVATE_KEY_PATH') }}`. On Windows that variable must be an absolute path with forward slashes, for example `C:/Users/you/keys/rsa_key.p8`. A backslash path is a YAML escape, and a relative path is resolved after dbt changes into the project directory.

dbt looks for `profiles.yml` in `~/.dbt/` by default (`%USERPROFILE%\.dbt\` on Windows). Either copy the file there or set `DBT_PROFILES_DIR` as above.

The profile name is `snowflake_sample_dbt`, matching `dbt_project.yml`.

There is no `packages.yml`. The composite-key test lives in this repo, so `dbt deps` is not required.

## Build

Use Python 3.10, 3.11, 3.12, or 3.13. Install dbt 1.8 or newer with the Snowflake adapter (`dbt-snowflake`). On Windows, activate the virtualenv and call `dbt` from this directory:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install "dbt-snowflake>=1.8"
dbt debug
dbt parse
dbt build
```

`dbt build` creates the eight staging views and runs the primary-key tests. Sources read `SNOWFLAKE_SAMPLE_DATA.TPCH_SF10`. The sample database ships these tables as `TPCH_SF1`, `TPCH_SF10`, `TPCH_SF100`, and `TPCH_SF1000`.
