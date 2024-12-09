---
title: "IBM watsonx/Presto configurations"
id: "watsonx-presto-configs"
---

## Cluster requirements

To use IBM watsonx.data or Presto with dbt, ensure the cluster has an attached catalog that allows creating, renaming, altering, and dropping objects such as tables and views. The user connecting to the cluster with dbt must have equivalent permissions for the target catalog.

## Session properties

With a IBM watsonx.data, or Presto cluster, you can [set session properties](https://prestodb.io/docs/current/sql/set-session.html) to modify the current configuration for your user session.

To temporarily adjust session properties for a specific dbt model or a group of models, use a [dbt hook](/reference/resource-configs/pre-hook-post-hook). For example:

```sql
{{
  config(
    pre_hook="set session query_max_run_time='10m'"
  )
}}
```

## Connector properties

IBM watsonx.data and Presto support various connector properties to manage how your data is represented. These properties are particularly useful for file-based connectors like Hive.

For details on what's supported for each supported data source, refer to either the [Presto Connectors](https://prestodb.io/docs/current/connector.html) or [watsonx.data Catalog](https://cloud.ibm.com/docs/watsonxdata?topic=watsonxdata-reg_database).


### Hive catalogs

When using the Hive connector with a metastore service (HMS), ensure the following settings are configured. These settings are crucial for enabling frequently executed operations like `DROP` and `RENAME` in dbt:

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
The `dbt-watsonx-presto` adapter offers comprehensive support for all [Presto datatypes](https://prestodb.io/docs/current/language/types.html) and [watsonx.data datatypes](https://www.ibm.com/support/pages/node/7157339) in seed files. However, to utilize this feature, you need to explicitly define the data types for each column in the `dbt_project.yml` file.

To configure column data types, update your `<project_name>/dbt_project.yml` file as follows:

```sh
seeds:
  <project_name>:
    <seed_file_name>:
      +column_types:
        <col_1>: <datatype>
        <col_2>: <datatype>
```
This ensures that dbt correctly interprets and applies the specified data types when loading seed data into your Presto or watsonx.data clusters.


## Materializations
### Table

The `dbt-watsonx-presto` adapter helps you create and update tables through table materialization, making it easier to work with data in Presto.

#### Recommendations
- **Check Permissions:** Ensure that the necessary permissions for table creation are enabled in the catalog or schema.
- **Check Connector Documentation:** Review Presto [connector’s documentation](https://prestodb.io/docs/current/connector.html) or watsonx.data [sql statement support](https://www.ibm.com/support/pages/node/7157339) to ensure it supports table creation and modification.

#### Limitations with Some Connectors
Certain Presto connectors, particularly read-only ones or those with restricted permissions, do not allow creating or modifying tables. If you attempt to use table materialization with these connectors, you may encounter an error like:

```sh
PrestoUserError(type=USER_ERROR, name=NOT_SUPPORTED, message="This connector does not support creating tables with data", query_id=20241206_071536_00026_am48r)
```

### View

The `dbt-watsonx-presto` adapter supports creating views through materialization. In Presto, the default view security mode is `DEFINER`, meaning that views execute with the permissions of the user who created them. If needed, this behavior can be adjusted to use `INVOKER` mode, where views execute with the permissions of the user running them.

#### Configuring View Security Modes in Presto
To modify the view security mode in `dbt-watsonx-presto`, you can use a **pre-hook** to set the **session property** `default_view_security_mode` before creating the view. For example, to use `INVOKER` mode, you can add the following configuration in your model:

```sql
{{
  config(
    materialized='view',
    pre_hook="SET SESSION default_view_security_mode='INVOKER'"
  )
}}
```

For more details about security modes in views, see [Security](https://prestodb.io/docs/current/sql/create-view.html#security) in the Presto docs. Additionally, consult the Presto [connector's documentation](https://prestodb.io/docs/current/connector.html) or watsonx.data [sql statement support](https://www.ibm.com/support/pages/node/7157339) to check if your connector supports view creation.


### Unsupported Features
The following features are not supported by the `dbt-watsonx-presto` adapter
- Incremental Materialization
- Materialized Views
- Snapshots
