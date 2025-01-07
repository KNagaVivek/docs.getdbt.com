---
title: "IBM watsonx.data Presto setup"
description: "Read this guide to learn about the IBM watsonx.data Presto setup in dbt."
id: "watsonx-presto setup"
meta:
  maintained_by: IBM
  authors: 
  github_repo: 'IBM/dbt-watsonx-presto'
  pypi_package: 'dbt-watsonx-presto'
  min_core_version: 1.8.5
  cloud_support: 'Not Supported'
  min_supported_version: 'n/a'
  slack_channel_name: 
  slack_channel_link: 
  platform_name: 
  config_page: 
---

The dbt-watsonx-presto adapter allows you to use dbt to transform and manage data on IBM watsonx.data Presto(Java), leveraging its distributed SQL query engine capabilities. The configuration and connection setup described here are also applicable to open-source Presto. Before proceeding, ensure you have the following:
<ul>
  <li>An active IBM watsonx.data Presto(Java) Engine with connection details (host, port, catalog, schema).</li>
  <li>Authentication Credentials: Username and password/apikey.</li>
  <li>For watsonx.data instances, SSL verification is required for secure connections. If the instance host uses HTTPS, there is no need to specify the SSL certificate parameter. However, if the instance host uses an unsecured HTTP connection, ensure you provide the path to the SSL certificate file.</li>
</ul>
Refer to the Configuring dbt-watsonx-presto section for guidance on obtaining and organizing these details.


<Snippet path="warehouse-setups-cloud-callout" />

import SetUpPages from '/snippets/_setup-pages-intro.md';

<SetUpPages meta={frontMatter.meta}/>


## Connecting to IBM watsonx.data Presto

To connect dbt with watsonx.data Presto(java), you need to configure a profile in your `profiles.yml` file located in the `.dbt/` directory of your home folder. The following is an example configuration for connecting to IBM watsonx.data SaaS and Software instances:

<File name='~/.dbt/profiles.yml'>

```yaml
my_project:
  outputs:
    software:
      type: presto
      method: BasicAuth
      user: [user]
      password: [password]
      host: [hostname]
      database: [database name]
      schema: [your dbt schema]
      port: [port number]
      threads: [1 or more]
      ssl_verify: path/to/certificate

    saas:
      type: presto
      method: BasicAuth
      user: [user]
      password: [api_key]
      host: [hostname]
      database: [database name]
      schema: [your dbt schema]
      port: [port number]
      threads: [1 or more]

  target: software

```

</File>

## Host parameters

The following profile fields are required for configuring watsonx.data Presto(java) connections. Currently, it supports only the `BasicAuth` authentication method. For IBM watsonx.data SaaS or Software instances, You can get the hostname and port details by clicking View connect details inside the Presto(java) engine details page.

| Option    | Required/Optional | Description | Example  |
| --------- | ------- | ------- | ----------- |
| `method`  | Required (default value is none) | Authentication method for Presto | `None` or `BasicAuth` |
|   `user`  | Required | Username or email for authentication. | `user` |
| `password`| Required (if `method` is `BasicAuth`) | Password or API key for authentication | `password` |
|   `host`  | Required | Hostname for connecting to Presto. | `127.0.0.1` |
| `database`| Required | The catalog name in your presto cluster. | `Analytics` |
|  `schema` | Required | The schema name within your presto cluster's catalog. | `my_schema`  |
|   `port`  | Required | Port for connecting to Presto.  | `443`  |
| ssl_verify | Optional (default: **true**) | Specifies the path to the SSL certificate or a boolean value. The SSL certificate path is required if the watsonx.data instance is not secure (HTTP).| `path/to/certificate` or `true` |


### Schemas and databases
When selecting the catalog and the schema, make sure the user has read and write access to both. This selection does not limit your ability to query the catalog. Instead, they serve as the default location for where tables and views are materialized. In addition, the Presto connector used in the catalog must support creating tables. This default can be changed later from within your dbt project.

### SSL Verification
- If the Presto instance uses an unsecured HTTP connection, you must set `ssl_verify` to the path of the SSL certificate file.
- If the instance uses `HTTPS`, this parameter is not required and can be omitted.

## Additional parameters

The following profile fields are optional to set up. They let you configure your cluster's session and dbt for your connection. 


| Profile field                 |  Description                                                                                                | Example                              |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `threads`                     | How many threads dbt should use (default is `1`)                                                            | `8`                                  |
| `http_headers`                | HTTP headers to send alongside requests to Presto, specified as a yaml dictionary of (header, value) pairs. | `X-Presto-Routing-Group: my-cluster` |
| `http_scheme`                 | The HTTP scheme to use for requests to    (default: `http`, or `https` if `BasicAuth`)                | `https` or `http`                    |

