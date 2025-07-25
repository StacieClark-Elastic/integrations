# SQL input

The SQL input package allows you to run custom queries against an SQL database and store the results in Elasticsearch.

This input package supports the following databases

- MySQL
- Oracle
- Microsoft SQL
- PostgreSQL

## Configuration options


### Hosts

The host configuration should be specified from where the metrics are to be fetched. It varies depending upon the driver you are running.

#### MySQL

The supported configuration takes this form
- `<user>:<password>@tcp(<host>:<port>)/`

Here is an example of the supported configuration:
- `root:root@tcp(localhost:3306)/`

#### Oracle 

Two types of host configurations are supported:

- Old style host configuration

    a. `hosts: ["user/pass@0.0.0.0:1521/ORCLPDB1.localdomain"]`
    b. `hosts: ["user/password@0.0.0.0:1521/ORCLPDB1.localdomain as sysdba"]`

- DSN host configuration

    a. `hosts: ['user="user" password="pass" connectString="0.0.0.0:1521/ORCLPDB1.localdomain"']`
    b. `hosts: ['user="user" password="password" connectString="host:port/service_name" sysdba=true']`
  
#### MSSQL

The supported configuration takes this form
- `sqlserver://<user>:<password>@<host>`

Here is an example of the supported configuration:
- `sqlserver://root:test@localhost`

#### PostgreSQL

The supported configuration takes this form
- `postgres://<user>:<password>@<connection_string>`

Here is an example of the supported configuration 
- `postgres://postgres:postgres@localhost:5432/stuff?sslmode=disable`

NOTE: If the password includes a backslash (\), you need to escape it by adding another backslash. For example, my\_password should be written as my\\_password.

### Driver

Specifies the driver for which you want to run the queries. These are the supported drivers:

- mysql
- oracle
- mssql
- postgres

### SQL_Queries

Receives the list of queries to run. `query` and `response_format` is repeated to get multiple query inputs.

For example:
```
sql_queries: 
  - query: SHOW GLOBAL STATUS LIKE 'Innodb_system%'
    response_format: variables
```

`response_format`: This can be either `variables` or `table`

- `variables`: Expects a two-column table that looks like a key/value result. The left column is considered a key and the right column the value. This mode generates a single event on each fetch operation.

- `table`: Expects any number of columns. This mode generates a single event for each row.

For more examples of response format please refer [here](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-sql.html)


### Merge Results
Merge multiple queries into a single event.

Multiple queries will create multiple events, one for each query.  It may be preferable to create a single event by combining the metrics together in a single event.

This feature can be enabled using the `merge_results` config.

`merge_results` can merge queries having response format as "variable". 
However, for queries with a response format as "table", a merge is possible only if each table query produces a single row.

For example, if we have the following queries for PostgreSQL:
```
sql_queries:
  - query: "SELECT blks_hit,blks_read FROM pg_stat_database LIMIT 1;"
    response_format: table
  - query: "SELECT checkpoints_timed,checkpoints_req FROM pg_stat_bgwriter;"
    response_format: table
```

The `merge_results` feature will create a combined event, where `blks_hit`, `blks_read`, `checkpoints_timed` and `checkpoints_req` are part of the same event.

### SSL configuration

The drivers `mysql`, `mssql`, and `postgres` are supported.

The SSL configuration is driver-specific. Different drivers have slightly different parameter interpretations. Subset of the [params](https://www.elastic.co/docs/reference/beats/metricbeat/configuration-ssl#ssl-client-config) is supported.

When "SSL Configuration" parameters are set, only URL-formatted connection strings are accepted.
Use this format: `postgres://myuser:mypassword@localhost:5432/mydb`.
Don't use this format: `user=myuser password=mypassword dbname=mydb`.

Example of SSL configuration:
```
verification_mode: full
certificate_authorities:
  - /path/to/ca.pem
```

#### `mysql` driver

Parameters supported: `verification_mode`, `certificate`, `key`, `certificate_authorities`.

The certificates can be passed both as file paths and certificate content.

Example with the certificate content "embedded":
```
verification_mode: full
certificate_authorities:
  - |
    -----BEGIN CERTIFICATE-----
    MIIDCjCCAfKgAwIBAgITJ706Mu2wJlKckpIvkWxEHvEyijANBgkqhkiG9w0BAQsF
    ADAUMRIwEAYDVQQDDAlsb2NhbGhvc3QwIBcNMTkwNzIyMTkyOTA0WhgPMjExOTA2
    MjgxOTI5MDRaMBQxEjAQBgNVBAMMCWxvY2FsaG9zdDCCASIwDQYJKoZIhvcNAQEB
    BQADggEPADCCAQoCggEBANce58Y/JykI58iyOXpxGfw0/gMvF0hUQAcUrSMxEO6n
    fZRA49b4OV4SwWmA3395uL2eB2NB8y8qdQ9muXUdPBWE4l9rMZ6gmfu90N5B5uEl
    94NcfBfYOKi1fJQ9i7WKhTjlRkMCgBkWPkUokvBZFRt8RtF7zI77BSEorHGQCk9t
    /D7BS0GJyfVEhftbWcFEAG3VRcoMhF7kUzYwp+qESoriFRYLeDWv68ZOvG7eoWnP
    PsvZStEVEimjvK5NSESEQa9xWyJOmlOKXhkdymtcUd/nXnx6UTCFgnkgzSdTWV41
    CI6B6aJ9svCTI2QuoIq2HxX/ix7OvW1huVmcyHVxyUECAwEAAaNTMFEwHQYDVR0O
    BBYEFPwN1OceFGm9v6ux8G+DZ3TUDYxqMB8GA1UdIwQYMBaAFPwN1OceFGm9v6ux
    8G+DZ3TUDYxqMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAG5D
    874A4YI7YUwOVsVAdbWtgp1d0zKcPRR+r2OdSbTAV5/gcS3jgBJ3i1BN34JuDVFw
    3DeJSYT3nxy2Y56lLnxDeF8CUTUtVQx3CuGkRg1ouGAHpO/6OqOhwLLorEmxi7tA
    H2O8mtT0poX5AnOAhzVy7QW0D/k4WaoLyckM5hUa6RtvgvLxOwA0U+VGurCDoctu
    8F4QOgTAWyh8EZIwaKCliFRSynDpv3JTUwtfZkxo6K6nce1RhCWFAsMvDZL8Dgc0
    yvgJ38BRsFOtkRuAGSf6ZUwTO8JJRRIFnpUzXflAnGivK9M13D5GEQMmIl6U9Pvk
    sxSmbIUfc2SGJGCJD4I=
    -----END CERTIFICATE-----
```

#### `postgres` driver

Parameters supported: `verification_mode`, `certificate`, `key`, `certificate_authorities`.

Only one certificate can be passed to the `certificate_authorities` parameter.
The certificates can be passed only as file paths. The files have to be present in the environment where the metricbeat is running.

The `verification_mode` is translated as follows:

- `full` -> `verify-full`

- `strict` -> `verify-full`

- `certificate` -> `verify-ca`

- `none` -> `require`

#### `mssql` driver

Params supported: `verification_mode`, `certificate_authorities`.

Only one certificate can be passed to the `certificate_authorities` parameter.
The certificates can be passed only as file paths. The files have to be present in the environment where the metricbeat is running.

If `verification_mode` is set to `none`, `TrustServerCertificate` will be set to `true`, otherwise it is `false`.


## Metrics reference

### Example

```json
{
    "@timestamp": "2025-06-25T07:34:08.850Z",
    "agent": {
        "ephemeral_id": "062e1a2d-efcc-495c-9cef-2f4d1ea6bdaa",
        "id": "81f6c307-e62b-45cd-aa0d-be554deb83b2",
        "name": "elastic-agent-33528",
        "type": "metricbeat",
        "version": "9.1.0"
    },
    "data_stream": {
        "dataset": "sql.sql",
        "namespace": "72095",
        "type": "metrics"
    },
    "ecs": {
        "version": "8.0.0"
    },
    "elastic_agent": {
        "id": "81f6c307-e62b-45cd-aa0d-be554deb83b2",
        "snapshot": true,
        "version": "9.1.0"
    },
    "event": {
        "agent_id_status": "verified",
        "dataset": "sql.sql",
        "duration": 1311560,
        "ingested": "2025-06-25T07:34:11Z",
        "module": "sql"
    },
    "host": {
        "architecture": "aarch64",
        "containerized": false,
        "hostname": "elastic-agent-33528",
        "ip": [
            "192.168.160.2",
            "172.28.0.4"
        ],
        "mac": [
            "02-42-AC-1C-00-04",
            "02-42-C0-A8-A0-02"
        ],
        "name": "elastic-agent-33528",
        "os": {
            "family": "",
            "kernel": "6.8.0-50-generic",
            "name": "Wolfi",
            "platform": "wolfi",
            "type": "linux",
            "version": "20230201"
        }
    },
    "metricset": {
        "name": "query",
        "period": 10000
    },
    "service": {
        "address": "svc-sql_input_mysql:3306",
        "type": "sql"
    },
    "sql": {
        "driver": "mysql",
        "metrics": {
            "delayed_insert_threads": "0",
            "mysqlx_worker_threads": "2",
            "mysqlx_worker_threads_active": "0",
            "slow_launch_threads": "0",
            "threads_cached": "0",
            "threads_connected": "1",
            "threads_created": "1",
            "threads_running": "2"
        },
        "query": [
            "SHOW STATUS LIKE '%Threads%'"
        ]
    }
}
```
