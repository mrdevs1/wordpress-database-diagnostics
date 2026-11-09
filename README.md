# WordPress Database Diagnostics

A practical guide to diagnosing, optimizing, and maintaining WordPress databases using WP-CLI, SQL queries, and built-in database tools.

## Why WordPress Database Maintenance Matters

WordPress stores posts, pages, users, settings, metadata, and plugin information in a MySQL or MariaDB database.

Over time, unnecessary data, inefficient queries, and poorly maintained tables can affect website performance and reliability.

## Common Database Problems

- Slow database queries
- Excessive database size
- Orphaned post metadata
- Expired transients
- Overloaded options table
- Missing database indexes
- Database connection errors
- Corrupted database tables

## 1. Check Database Information

Use WP-CLI to inspect database configuration:

```bash
wp db check
```

Check database size:

```bash
wp db size
```

Check database tables:

```bash
wp db tables
```

These commands help identify database health and storage-related issues.

## 2. Repair Database Tables

WordPress provides a database repair mechanism.

Add the following to `wp-config.php` temporarily:

```php
define('WP_ALLOW_REPAIR', true);
```

Visit:

```text
https://example.com/wp-admin/maint/repair.php
```

Run the available repair option.

Remove the configuration constant after completing maintenance. Restrict access to this page while it is enabled.

## 3. Optimize Database Tables

Use WP-CLI to optimize database tables:

```bash
wp db optimize
```

Alternatively, use SQL:

```sql
OPTIMIZE TABLE wp_posts;
OPTIMIZE TABLE wp_postmeta;
OPTIMIZE TABLE wp_options;
```

Replace `wp_` with your actual database table prefix.

Optimization benefits depend on the database engine and table condition.

## 4. Identify Large Database Tables

Run the following SQL query:

```sql
SELECT
    table_name,
    ROUND((data_length + index_length) / 1024 / 1024, 2) AS size_mb
FROM information_schema.tables
WHERE table_schema = DATABASE()
ORDER BY (data_length + index_length) DESC;
```

This identifies the largest tables and helps prioritize database investigation.

## 5. Find Expired Transients

Expired transients can accumulate in the WordPress options table.

Inspect them using WP-CLI:

```bash
wp transient list
```

Delete expired transients:

```bash
wp transient delete --expired
```

Avoid deleting all transients without understanding their purpose, as plugins may need to regenerate cached data.

## 6. Identify Orphaned Post Metadata

Orphaned metadata refers to post metadata associated with posts that no longer exist.

Find orphaned metadata:

```sql
SELECT pm.meta_id, pm.post_id, pm.meta_key
FROM wp_postmeta pm
LEFT JOIN wp_posts p ON pm.post_id = p.ID
WHERE p.ID IS NULL;
```

Review the results before deleting anything.

If verified as unnecessary, remove orphaned records:

```sql
DELETE pm
FROM wp_postmeta pm
LEFT JOIN wp_posts p ON pm.post_id = p.ID
WHERE p.ID IS NULL;
```

Always back up the database before executing destructive SQL queries.

## 7. Investigate Autoloaded Options

Autoloaded options are loaded during WordPress initialization.

Inspect their total size:

```sql
SELECT
    SUM(LENGTH(option_value)) AS total_bytes
FROM wp_options
WHERE autoload IN ('yes', 'on', 'auto-on', 'auto');
```

Find the largest autoloaded options:

```sql
SELECT
    option_name,
    LENGTH(option_value) AS size_bytes
FROM wp_options
WHERE autoload IN ('yes', 'on', 'auto-on', 'auto')
ORDER BY size_bytes DESC
LIMIT 20;
```

Large autoloaded options can increase the amount of data loaded on every request.

Do not disable autoloading or delete options without identifying which plugin or theme owns them.

## 8. Investigate Slow Database Queries

Enable WordPress debugging in a controlled development environment:

```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

Review:

```text
wp-content/debug.log
```

For deeper investigation, use MySQL's slow query log and `EXPLAIN` to understand query execution plans.

Example:

```sql
EXPLAIN SELECT *
FROM wp_posts
WHERE post_status = 'publish';
```

Look for inefficient table scans, missing indexes, and unnecessarily expensive queries.

## 9. Check Database Connection

Use WP-CLI to verify database connectivity:

```bash
wp db check
```

If database connection errors occur, investigate:

- Database server availability
- Database hostname
- Database username and password
- Database permissions
- Connection limits
- Server resource availability

Avoid changing database credentials without verifying the hosting configuration.

## 10. Essential Database Maintenance Commands

| Command | Purpose |
|---|---|
| `wp db check` | Check database tables |
| `wp db size` | Check database size |
| `wp db tables` | List database tables |
| `wp db optimize` | Optimize database tables |
| `wp transient list` | List transients |
| `wp transient delete --expired` | Delete expired transients |
| `wp db export backup.sql` | Export database backup |

## Database Troubleshooting Checklist

- [ ] Create a database backup.
- [ ] Check database health.
- [ ] Identify unusually large tables.
- [ ] Investigate slow queries.
- [ ] Review autoloaded options.
- [ ] Inspect expired transients.
- [ ] Identify orphaned metadata.
- [ ] Review database indexes.
- [ ] Optimize tables when appropriate.
- [ ] Verify website functionality after maintenance.

## Important Safety Guidelines

1. Always create a verified backup before database modifications.
2. Test destructive SQL queries on staging first.
3. Never delete unknown plugin or theme data.
4. Avoid unnecessary database optimization.
5. Use database indexes carefully.
6. Monitor performance before and after maintenance.

## Conclusion

WordPress database maintenance is essential for website performance, reliability, and long-term stability.

WP-CLI, SQL queries, and database monitoring tools provide an effective approach to diagnosing database problems and performing safe maintenance.

This repository serves as a practical reference for WordPress developers and administrators.
