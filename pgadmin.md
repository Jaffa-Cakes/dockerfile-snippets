# pgAdmin

Some notes on [`dpage/pgadmin4`](https://hub.docker.com/r/dpage/pgadmin4/) images.

The full documentation is [here](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html).

## Table of Contents

- [pgAdmin](#pgadmin)
  - [Table of Contents](#table-of-contents)
  - [Essential environment variables](#essential-environment-variables)
  - [Removing login screen](#removing-login-screen)
  - [Removing master password](#removing-master-password)
  - [Adding servers](#adding-servers)

## Essential environment variables

```yaml
PGADMIN_DEFAULT_EMAIL: 'user@example.com'
PGADMIN_DEFAULT_PASSWORD: 'password'
```

## Removing login screen

The pgAdmin container comes with a login screen by default, this can by bypassed by setting the following enviroment variables:

```yaml
PGADMIN_CONFIG_SERVER_MODE: 'False'
```

## Removing master password

The pgAdmin container comes with a master password by default, this can by bypassed by setting the following enviroment variables:

```yaml
PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED: 'False'
```

## Adding servers

Servers can be added by overridding the [`servers.json`](https://www.pgadmin.org/docs/pgadmin4/development/import_export_servers.html#json-format) file at `/pgadmin4/servers.json` with:

```json
{
    "Servers": {
        "1": {
            "Name": "Server Name",
            "Group": "Servers",
            "Port": 5432,
            "Username": "postgres",
            "Host": "db",
            "MaintenanceDB": "postgres",
            "SSLMode": "prefer",
            "PassFile": "/pgpass" // Omit if not using password authentication
        }
    }
}
```

If you are using password authentication, you will need to mount a [`pgpass`](https://www.postgresql.org/docs/current/libpq-pgpass.html) file into the container at `/pgpass` with the following format:

```yaml
# hostname:port:database:username:password
db:5432:postgres:postgres:postgres
```