---
title: "IBM watsonx.data Presto configurations"
id: "watsonx-presto-configs"
---

## Instance requirements

To use IBM watsonx.data Presto(java) with dbt, ensure the instance has an attached catalog that allows creating, renaming, altering, and dropping objects such as tables and views. The user connecting to the instance with dbt must have equivalent permissions for the target catalog.

## Session properties

With IBM watsonx.data SaaS/Software, or Presto instance, you can [set session properties](https://prestodb.io/docs/current/sql/set-session.html) to modify the current configuration for your user session.

To temporarily adjust session properties for a specific dbt model or a group of models, use a [dbt hook](/reference/resource-configs/pre-hook-post-hook). For example:

```sql
{{
  config(
    pre_hook="set session query_max_run_time='10m'"
  )
}}
```

## Connector properties

IBM watsonx.data SaaS/Software and Presto support various connector properties to manage how your data is represented. These properties are particularly useful for file-based connectors like Hive.

For information on what is supported for each data source, refer to one of the following resources:
- [Presto Connectors](https://prestodb.io/docs/current/connector.html)
- [watsonx.data SaaS Catalog](https://cloud.ibm.com/docs/watsonxdata?topic=watsonxdata-reg_database)
- [watsonx.data Software Catalog](https://www.ibm.com/docs/en/watsonx/watsonxdata/1.1.x?topic=components-adding-database-catalog-pair)


### Hive catalogs

When using the Hive connector, ensure the following settings are configured. These settings are crucial for enabling frequently executed operations like `DROP` and `RENAME` in dbt:

```java
hive.metastore-cache-ttl=0s
hive.metastore-refresh-interval=5s
hive.allow-drop-table=true
hive.allow-rename-table=true

```

## File format configuration

For file-based connectors, such as Hive, you can customize table materialization and data formats. For example, to create a partitioned [Parquet](https://spark.apache.org/docs/latest/sql-data-sources-parquet.html) table:

```sql
{{
  config(
    materialized='table',
    properties={
      "format": "'PARQUET'",
      "partitioning": "ARRAY['bucket(id, 2)']",
    }
  )
}}
```

## Seeds and prepared statements
The `dbt-watsonx-presto` adapter offers comprehensive support for all [Presto datatypes](https://prestodb.io/docs/current/language/types.html) and [watsonx.data Presto datatypes](https://www.ibm.com/support/pages/node/7157339) in seed files. However, to utilize this feature, you need to explicitly define the data types for each column in the `dbt_project.yml` file.

To configure column data types, update your `<project_name>/dbt_project.yml` file as follows:

```sh
seeds:
  <project_name>:
    <seed_file_name>:
      +column_types:
        <col_1>: <datatype>
        <col_2>: <datatype>
```
This ensures that dbt correctly interprets and applies the specified data types when loading seed data into your watsonx.data Presto instances.


## Materializations
### Table

The `dbt-watsonx-presto` adapter helps you create and update tables through table materialization, making it easier to work with data in watsonx.data Presto.

#### Recommendations
- **Check Permissions:** Ensure that the necessary permissions for table creation are enabled in the catalog or schema.
- **Check Connector Documentation:** Review Presto [connector’s documentation](https://prestodb.io/docs/current/connector.html) or watsonx.data Presto [sql statement support](https://www.ibm.com/support/pages/node/7157339) to ensure it supports table creation and modification.

#### Limitations with Some Connectors
Certain watsonx.data Presto connectors, particularly read-only ones or those with restricted permissions, do not allow creating or modifying tables. If you attempt to use table materialization with these connectors, you may encounter an error like:

```sh
PrestoUserError(type=USER_ERROR, name=NOT_SUPPORTED, message="This connector does not support creating tables with data", query_id=20241206_071536_00026_am48r)
```

### View

The `dbt-watsonx-presto` adapter supports creating views using the `materialized='view'` configuration in your dbt model. By default, when you set the materialization to view, it creates a view in watsonx.data Presto.

```sql
{{
  config(
    materialized='view',
  )
}}
```

For more details, refer to the watsonx.data [sql statement support](https://www.ibm.com/support/pages/node/7157339) or Presto [connector documentation](https://prestodb.io/docs/current/connector.html) to verify whether your connector supports view creation.


### Unsupported Features
The following features are not supported by the `dbt-watsonx-presto` adapter
- Incremental Materialization
- Materialized Views
- Snapshots
