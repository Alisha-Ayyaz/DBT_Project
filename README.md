# Airbnb Analytics dbt Project

A dbt and Snowflake data pipeline that transforms Airbnb listing, host, and booking data into analytics-ready models. The project organizes transformations into bronze, silver, and gold layers, with snapshot models for tracking changes over time.

## Pipeline overview

The dbt project is located in [`aws_dbt_snowflake_project/`](./aws_dbt_snowflake_project/).

| Layer | Purpose | Models |
|---|---|---|
| Staging source | Defines the Snowflake staging tables consumed by dbt | `AIRBNB.STAGING.LISTINGS`, `HOSTS`, `BOOKINGS` |
| Bronze | Loads source records and incrementally processes new records | `bronze_listings`, `bronze_hosts`, `bronze_bookings` |
| Silver | Applies business transformations and prepares clean entity-level data | `silver_listings`, `silver_hosts`, `silver_bookings` |
| Gold | Produces joined and reporting-oriented datasets | `obt`, `fact` |
| Snapshots | Preserves historical versions of booking, host, and listing records | `dim_bookings`, `dim_hosts`, `dim_listings` |

The project also includes reusable SQL macros and ephemeral models for intermediate transformations.

## Requirements

- Python 3.12 or later
- [uv](https://docs.astral.sh/uv/)
- Access to a Snowflake account with permissions to read the configured staging tables and create or update dbt relations

## Setup

From the repository root, install the Python dependencies:

```powershell
uv sync
```

Configure a dbt profile named `aws_dbt_snowflake_project` in `%USERPROFILE%\.dbt\profiles.yml`. Use your Snowflake account, user, role, warehouse, and database values. Keep the password out of the profile file; reference an environment variable instead:

```yaml
aws_dbt_snowflake_project:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: YOUR_SNOWFLAKE_ACCOUNT
      user: YOUR_SNOWFLAKE_USER
      password: "{{ env_var('DBT_ENV_SECRET_SNOWFLAKE_PASSWORD') }}"
      role: YOUR_SNOWFLAKE_ROLE
      warehouse: YOUR_SNOWFLAKE_WAREHOUSE
      database: AIRBNB
      schema: dbt_schema
      threads: 1
```

In PowerShell, set the password for the current terminal session without adding it to the repository:

```powershell
$env:DBT_ENV_SECRET_SNOWFLAKE_PASSWORD = "your-password"
```

Activate the virtual environment if desired, then change into the dbt project directory:

```powershell
.\.venv\Scripts\Activate.ps1
cd .\aws_dbt_snowflake_project
```

## Run the pipeline

Validate the connection and profile:

```powershell
dbt debug
```

Build the models and run configured snapshots:

```powershell
dbt build
```

You can also run only a particular resource type:

```powershell
dbt run
dbt snapshot
```

Run these commands from `aws_dbt_snowflake_project`, where `dbt_project.yml` is located. If you remain at the repository root, pass the project path explicitly, for example:

```powershell
dbt build --project-dir .\aws_dbt_snowflake_project
```

## Repository layout

```text
.
├── aws_dbt_snowflake_project/
│   ├── macros/
│   ├── models/
│   │   ├── bronze/
│   │   ├── silver/
│   │   ├── gold/
│   │   └── sources/
│   ├── snapshots/
│   └── tests/
├── src/
├── pyproject.toml
└── uv.lock
```

Do not commit passwords, tokens, or other credentials. Use the `DBT_ENV_SECRET_SNOWFLAKE_PASSWORD` environment variable for the Snowflake password.
