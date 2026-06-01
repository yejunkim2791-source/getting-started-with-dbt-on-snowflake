# Local dbt Setup

This repository contains the dbt project in `tasty_bytes_dbt_demo`.

## 1. Install Python and dbt

Install Python 3.11 or 3.12, then install the Snowflake dbt adapter:

```powershell
py -m pip install --upgrade pip
py -m pip install dbt-snowflake
```

Confirm the install:

```powershell
dbt --version
```

If `dbt` is not found until a new terminal is opened, run it directly from:

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\dbt.exe" --version
```

## 2. Configure Snowflake credentials

Copy `.env.example` to `.env` and fill in your Snowflake values:

```powershell
Copy-Item .env.example .env
notepad .env
```

Then load the values into your current PowerShell session:

```powershell
Get-Content .env | ForEach-Object {
  if ($_ -match '^\s*([^#][^=]+)=(.*)$') {
    [Environment]::SetEnvironmentVariable($matches[1].Trim(), $matches[2].Trim(), 'Process')
  }
}
```

## 3. Run dbt locally

Use the local profile directory that is ignored by Git:

```powershell
cd .\tasty_bytes_dbt_demo
$env:DBT_PROFILES_DIR = '..\.dbt'
dbt debug
dbt deps
dbt build
```

If the current terminal does not resolve `dbt`, replace `dbt` with:

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\dbt.exe"
```

## 4. Snowflake objects

Before running `dbt build`, execute `tasty_bytes_dbt_demo/setup/tasty_bytes_setup.sql` in Snowflake to create and load:

- `TASTY_BYTES_DBT_DB`
- `RAW`, `DEV`, and `PROD` schemas
- `TASTY_BYTES_DBT_WH`
- source tables used by the dbt models
