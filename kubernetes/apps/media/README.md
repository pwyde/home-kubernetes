# Arr Applications

## Migrating Data from SQLite to PostgreSQL Databases

This guide will use **Radarr** as an example. Same procedure can be applied to all the **Arr** applications.

- **Radarr**
- **Sonarr**
- **Lidarr**
- **Prowlarr**

Upgrade **Radarr** to at least `v4.1.0.6133` or newer. This brings in support for **PostgreSQL**. This will also ensure that all of the **SQLite** tables have the latest schema migrations applied before migration to the PostgreSQL cluster.

Copy the SQLite databases from the Radarr container/pod, both the **main** database and **logs** database. This guide will assume the files are named `radarr.db` and `logs.db`.

Deploy Radarr in the Kubernetes cluster. Ensure that all schema migrations have been applied using the container/pod logs.

Restart Radarr using the web UI and double-check that connection to PostgreSQL can be re-established and that no other schema migrations is performed. Check the container/pod logs for this

Remove Radarr deployment in the Kubernetes cluster.

Dump the schema for the **main** and **logs** databases using `pg_dump` with the default superuser login.

```sh
pg_dump -h postgres.${SECRET_DOMAIN} -U postgres -s -d radarr_main -f /home/${USER}/radarr_main.sql
pg_dump -h postgres.${SECRET_DOMAIN} -U postgres -s -d radarr_main -f /home/${USER}/radarr_log.sql
```

In order to reset all tables to a clean starting point before importing the SQLite data, drop and re-create the databases on the PostgreSQL cluster. Using `psql` with the default superuser login.

```sh
psql -h postgres.${SECRET_DOMAIN} -U postgres
```

```
DROP DATABASE "radarr_main";
DROP DATABASE "radarr_log";
CREATE DATABASE "radarr_main";
CREATE DATABASE "radarr_log";
ALTER SCHEMA public OWNER TO radarr;
ALTER DATABASE "radarr_main" OWNER TO radarr;
ALTER DATABASE "radarr_log" OWNER TO radarr;
\q
```

The databases are now re-created. Now the schema must be re-imported that was dumped earlier.

```sh
psql -h postgres.${SECRET_DOMAIN} -U radarr -d radarr_main -f ${HOME}/radarr_main.sql
psql -h postgres.${SECRET_DOMAIN} -U radarr -d radarr_log -f ${HOME}/radarr_log.sql
```

Now with the schema re-imported and all tables clean, the SQLite databases can be migrated using `pgloader`.

```sh
docker run --rm -v ${HOME}/radarr.db:/radarr.db:ro --network=host ghcr.io/roxedus/pgloader --with "quote identifiers" --with "data only" /radarr.db "postgresql://radarr:${RADARR__POSTGRES__PASSWORD}@postgres.${SECRET_DOMAIN}/radarr_main"
docker run --rm -v ${HOME}/logs.db:/logs.db:ro --network=host ghcr.io/roxedus/pgloader --with "quote identifiers" --with "data only" /logs.db "postgresql://radarr:${RADARR__POSTGRES__PASSWORD}@postgres.${SECRET_DOMAIN}/radarr_log"
```

Re-deploy Radarr in the Kubernetes cluster.
