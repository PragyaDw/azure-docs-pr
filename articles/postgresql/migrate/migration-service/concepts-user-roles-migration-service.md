---
title: "Migration service - Migration of users/roles, ownerships, and privileges"
description: Migration of users/roles, ownerships, and privileges along with schema and data
author: shriramm
ms.author: shriramm
ms.reviewer: maghan
ms.date: 06/19/2024
ms.service: postgresql
ms.topic: conceptual
---

# Migration of user roles, ownerships, and privileges for the migrations service in Azure Database for PostgreSQL

[!INCLUDE [applies-to-postgresql-flexible-server](~/reusable-content/ce-skilling/azure/includes/postgresql/includes/applies-to-postgresql-flexible-server.md)]

> [!IMPORTANT]  
> The migration of user roles, ownerships, and privileges feature is available only for the Azure Database for PostgreSQL Single server as the source. This feature is currently disabled for PostgreSQL version 16 servers.

The migration service automatically provides the following built-in capabilities for the Azure Database for PostgreSQL single server as the source and data migration.

- Migration of user roles on your source server to the target server.
- Migration of ownership of all the database objects on your source server to the target server.
- Migration of permissions of database objects on your source server, such as GRANTS/REVOKES, to the target server.

## Permission differences between Azure Database for PostgreSQL Single server and Flexible server
This section explores the differences in permissions granted to the **azure_pg_admin** role across single server and flexible server environments.

### PG catalog permissions
Unlike user-created schemas, which organize database objects into logical groups, pg_catalog is a system schema. It houses crucial system-level information, such as details about tables, columns, and other internal bookkeeping data. Essentially, it’s where PostgreSQL stores important metadata.

In a single server environment, a user belonging to the azure_pg_admin role is granted select privileges for all pg_catalog tables and views. However, in a flexible server, we restricted privileges for certain tables and views, allowing only the super user to query them. 

We removed all privileges for non-superusers on the following pg_catalog tables. 
- pg_authid 

- pg_largeobject 

- pg_statistic

- pg_subscription 

- pg_user_mapping 

We removed all privileges for non-superusers on the following pg_catalog views.
- pg_config 

- pg_file_settings 

- pg_hba_file_rules 

- pg_replication_origin_status 

- pg_shadow 

Allowing unrestricted access to these system tables and views could lead to unauthorized modifications, accidental deletions, or even security breaches. By restricting access, we're reducing the risk of unintended changes or data exposure. 

### pg_pltemplate deprecation

Another important consideration is the deprecation of the **pg_pltemplate** system table within the pg_catalog schema by the PostgreSQL community **starting from version 13.** Therefore, if you're migrating to Flexible Server versions 13 and above, and if you have granted permissions to users on the pg_pltemplate table, it is necessary to undo these permissions before initiating the migration process.

#### What is the impact?
- If your application is designed to directly query the affected tables and views, it will encounter issues upon migrating to the flexible server. We strongly advise you to refactor your application to avoid direct queries to these system tables. 

- If you have specifically granted or revoked privileges to any users or roles for the affected pg_catalog tables and views, you will encounter an error during the migration process. This error will be identified by the following pattern: 

```sql
pg_restore error: could not execute query <GRANT/REVOKE> <PRIVILEGES> on <affected TABLE/VIEWS> to <user>.
 ```

To resolve this error, it's necessary to undo the privileges granted to users and roles on the affected pg_catalog tables and views. You can accomplish this by taking the following steps.

   **Step 1: Identify Privileges**

Execute the following query on your single server by logging in as the admin user:

```sql
SELECT
  array_to_string(array_agg(acl.privilege_type), ', ') AS privileges,
  t.relname AS relation_name, 
  r.rolname AS grantee
FROM
  pg_catalog.pg_class AS t
  CROSS JOIN LATERAL aclexplode(t.relacl) AS acl
  JOIN pg_roles r ON r.oid = acl.grantee
WHERE
  acl.grantee <> 'azure_superuser'::regrole
  AND t.relname IN (
    'pg_authid', 'pg_largeobject', 'pg_subscription', 'pg_user_mapping', 'pg_statistic',
    'pg_config', 'pg_file_settings', 'pg_hba_file_rules', 'pg_replication_origin_status', 'pg_shadow', 'pg_pltemplate'
  )
GROUP BY
  r.rolname, t.relname;

```

**Step 2: Review the Output**

The output of the above query will show the list of privileges granted to roles on the impacted tables and views.

For example:

| Privileges | Relation name | Grantee |
| :--- | :--- | :--- | 
| SELECT | pg_authid | adminuser1
| SELECT, UPDATE | pg_shadow | adminuser2

**Step 3: Undo the privileges**

To undo the privileges, run REVOKE statements for each privilege on the relation from the grantee. In the above example, you would run:

```sql
REVOKE SELECT ON pg_authid FROM adminuser1;
REVOKE SELECT ON pg_shadow FROM adminuser2;
REVOKE UPDATE ON pg_shadow FROM adminuser2;
```

After completing these steps, you can proceed to initiate a new migration from the single server to the flexible server using the migration service. You should not encounter permission-related issues during this process.

## Related content
- [Migration service](concepts-migration-service-postgresql.md)
- [Known issues and limitations](concepts-known-issues-migration-service.md)
- [Network setup](how-to-network-setup-migration-service.md)
