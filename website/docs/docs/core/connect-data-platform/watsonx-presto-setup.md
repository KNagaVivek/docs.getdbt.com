---
title: "IBM/watsonx Presto setup"
description: "Read this guide to learn about the IBM/watsonx Presto setup in dbt."
id: "watsonx-presto setup"
meta:
  maintained_by: IBM
  authors: 
  github_repo: 'IBM/dbt-watsonx-presto'
  pypi_package: 'dbt-watsonx-presto'
  min_core_version: 
  cloud_support: 'Not Supported'
  min_supported_version: 'n/a'
  slack_channel_name: 
  slack_channel_link: 
  platform_name: 
  config_page: 
---


The dbt-presto adapter allows you to use dbt to transform and manage data on IBM Watsonx Presto, leveraging its distributed SQL query engine capabilities. Before proceeding, ensure you have the following:
<ul>
  <li>An active IBM watsonx Presto Engine or Presto setup with connection details (host, port, catalog, schema).</li>
  <li>Authentication Credentials: Username and password.</li>
  <li>SSL Certificate of the host: SSL verification is required for wxd on-prem clusters, ensure you have the path to the certificate file.</li>
</ul>
Refer to the Configuring dbt-presto section for guidance on obtaining and organizing these details.


<Snippet path="warehouse-setups-cloud-callout" />

import SetUpPages from '/snippets/_setup-pages-intro.md';

<SetUpPages meta={frontMatter.meta}/>


## Connecting to IBM watsonx/presto

To connect dbt Core with Presto, you need to configure a profile in your profiles.yml file located in the .dbt/ directory of your home folder. Below is an example configuration:

The parameters for setting up a connection are for IBM watsonx.data clusters. Unless specified, "cluster" will mean any of these products' clusters.


<File name='~/.dbt/profiles.yml'>

```yaml
my_project:
  outputs:
    dev:
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
  target: dev

```

</File>

## Host parameters

The following profile fields are always required except for `user`, which is also required unless you're using the `oauth`, `oauth_console`, `cert`, or `jwt` authentication methods.
Presto supports only BasicAuth as method currenlty. For IBM watsonx.data, You can get the hostname and port details by clicking View connect details inside the presto engine details page.

| Option    | Required/Optional | Description | Example  |
| --------- | ------- | ------- | ----------- |
| `method`  | Optional (default value is none) | Authentication method for Presto | `None` or `BasicAuth` |
|   `user`  | Required | Username for authentication. | `user` |
| `password`| Required (if `method` is `BasicAuth`) | Password or API key for authentication | `password` |
|   `host`  | Required | Hostname for connecting to Presto. | `127.0.0.1` |
| `database`| Required | The catalog name in your presto cluster. | `Analytics` |
|  `schema` | Required | The schema name within your presto cluster's catalog. | `my_schema`  |
|   `port`  | Required | Port for connecting to Presto.  | `443`  |

Note: For IBM Watsonx Presto cluster, the hostname and port details can be obtained from the "View Connect Details" section of the Presto engine details page.

### Schemas and databases
When selecting the catalog and the schema, make sure the user has read and write access to both. This selection does not limit your ability to query the catalog. Instead, they serve as the default location for where tables and views are materialized. In addition, the Trino connector used in the catalog must support creating tables. This default can be changed later from within your dbt project.

### SSL Verification
If the Presto instance uses SSL, set ssl_verify to the path of the certificate file.
If SSL is not required, this parameter can be omitted.

## Additional parameters

The following profile fields are optional to set up. They let you configure your cluster's session and dbt for your connection. 


| Profile field                 |  Description                                                                                                | Example                              |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `threads`                     | How many threads dbt should use (default is `1`)                                                            | `8`                                  |
| `http_headers`                | HTTP headers to send alongside requests to Presto, specified as a yaml dictionary of (header, value) pairs. | `X-Presto-Routing-Group: my-cluster` |
| `http_scheme`                 | The HTTP scheme to use for requests to PResto   (default: `http`, or `https` if `BasicAuth`)                | `https` or `http`                    |

## Limitations

### Transaction mode
Due to the nature of Presto, not all core dbt functionality is supported. The following features of dbt are not implemented on Presto:

Archival
Incremental models
