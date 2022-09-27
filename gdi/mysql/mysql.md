(mysql)=
# MySQL
<meta name="description" content="Documentation on the MySQL monitor">

## Description

The Splunk Distribution of OpenTelemetry Collector provides this integration as the MySQL monitor type using the Smart Agent Receiver. 
Use this integration to retrieve metrics and logs from MySQL.

This monitor connects to a MySQL instance and reports on the values returned by a `SHOW STATUS` command, which include the following:

  - Number of commands processed
  - Table and row operations (handlers)
  - State of the query cache
  - Status of MySQL threads
  - Network traffic

```{note}
This monitor is not available on Windows.
```

### Benefits

```{include} /_includes/benefits.md
```

## Installation

```{include} /_includes/collector-installation-linux.md
```

### Creating a MySQL user for this monitor

To create a MySQL user for this monitor, run the following commands:

```sql
 CREATE USER '<username>'@'localhost' IDENTIFIED BY '<password>';
 -- Give appropriate permissions
 -- ("GRANT USAGE" is synonymous to "no privileges")
 GRANT USAGE ON *.* TO '<username>'@'localhost';
 -- Permissions for the stats options
 GRANT REPLICATION CLIENT ON *.* TO '<username>'@'localhost';
```

The new user only has enough privileges to connect to the database. Additional privileges are not required.

### Considerations on localhost

For connections to `localhost`, MySQL programs attempt to connect to the local server by using a Unix socket file. To ensure that the client makes a TCP/IP connection to the local server specify a host name value of `127.0.0.1`, or the IP address or name of the local server.

## Configuration

```{include} /_includes/configuration.md
```

```{note}
Provide a MySQL monitor entry in your Smart Agent or Collector configuration. Use the appropriate form for your agent type.
```

### Splunk Distribution of OpenTelemetry Collector

To activate this monitor in the Splunk Distribution of OpenTelemetry Collector, add the following to your agent configuration. For example:

```yaml
receivers:
  smartagent/mysql:
    type: collectd/mysql
    host: 127.0.0.1
    port: 3306
    username: <global-username-for-all-db>
    password: <global-password-for-all-db>
    databases:
      - name: <name-of-db>
        username: <username> #Overrides global username
        password: <password> #Overrides global password
```

The following is a sample YAML configuration that shows how to connect multiple MySQL databases:

```yaml
receivers:
  smartagent/mysql:
    type: collectd/mysql
    host: 127.0.0.1
    port: 3306
    databases:
      - name: <name>
        username: <username>
        password: <password>
      - name: <name>
        username: <username>
        password: <password>
```

To complete the monitor activation, you must also include the `smartagent/mysql` receiver item in a `metrics` pipeline. To do this, add the receiver item to the `service` > `pipelines` > `metrics` > `receivers` section of your configuration file. For example:

```yaml
service:
  pipelines:
    metrics:
      receivers: [smartagent/mysql]
    logs:
      receivers: [smartagent/mysql]
```

### Smart Agent

To activate this monitor in the Smart Agent, add the following to your agent configuration:

```yaml
monitors:  # All monitor config goes under this key
 - type: collectd/mysql
   ...  # Additional config
```

The following is a sample YAML configuration that shows how to connect multiple MySQL databases:

```yaml
monitors:
 - type: collectd/mysql
   host: 127.0.0.1
   port: 3306
   databases:
     - name: <name-of-db>
     - name: <name-of-db>
       username: <username>
       password: <password>
   username: <global-username>
   password: <global-password>
```

The following is a sample YAML configuration that shows how to connect a single MySQL database:

```yaml
monitors:
 - type: collectd/mysql
    host: 127.0.0.1
    port: 3306
    databases:
      - name:
    username: <username>
    password: <password>
```

See {ref}`smart-agent` for an autogenerated example of a YAML configuration file, with default values where applicable.

### Configuration settings

The following table shows the configuration options for this monitor:

| Option | Required | Type | Description |
| --- | --- | --- | --- |
| `host` | Yes | `string` | Hostname or IP address of the MySQL instance. For example, `127.0.0.1`. |
| `port` | Yes | `integer` | The port of the MySQL instance. For example, `3306`. |
| `databases` | Yes | `list of objects` | A list of databases along with optional authentication credentials. |
| `username` | No | `string` | Username for all databases. You can override it by defining each username in the `databases` object. |
| `password` | No | `string` | Password for all databases. You can override it by defining each username in the `databases` object. |
| `reportHost` | No | `bool` | When set to `true`, the `host` dimension is set to the name of the MySQL database host. When `false`, the monitor uses the global `hostname` configuration instead. The default value is `false`. When `disableHostDimensions` is set to `true`, the host name in which the agent or monitor is running is not used for the `host` metric dimension value.  |
| `innodbStats` | No | `bool` | Collects InnoDB statistics. Before enabling InnoDB metrics make sure that you granted the `PROCESS` privilege to your user. The default value is `false`. |

The nested `databases` configuration object has the following fields:

| Option | Required | Type | Description |
| --- | --- | --- | --- |
| `name` | Yes | `string` | Name of the database. |
| `username` | No | `string` | Username of the database. |
| `password` | No | `string` | Password of the database. |

## Metrics

The following metrics are available for this integration:

<div class="metrics-yaml" url="https://raw.githubusercontent.com/signalfx/integrations/master/mysql/metrics.yaml"></div>

## Get help

```{include} /_includes/troubleshooting.md
```