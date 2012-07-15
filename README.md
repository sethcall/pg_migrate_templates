pg_migrate_templates
====================

A repository of just the sql 'wrappers' in template form, so that downstream pg_migrate projects can limit actual code.

The wrappers use plpgsql liberally.

Overview
--------
These templates follow .erb style syntax, but are purposefully without anything but string substitution, meaning any language could readily do a string replacement on the few locations where <%= var %> syntax is used.

bootstrap.erb
-------------
This template defines a transaction block of SQL which should be run before every migration attempt.  bootstrap.erb performs the following:
* Creates `FUNCTION bootstrap_pg_migrate`, which creates the pg_migrate and pg_migrations tables, if they don't exist.
* Creates `FUNCTION verify_against_existing_migrations`, which verifies that an existing migration of a certain name also has the correct ordinal position in the list of migrations.
* Creates `FUNCTION bypass_existing_migration`, which performs a `RAISE EXCEPTION` with a very particular exception message, so that code callers know tthis particular exception is not fatal; it just means move on to the next migration.  This function is called just before the user's migration code is executed (per migration).
* Creates `FUNCTION record_migration`, which manages rows in the pg_migrate and pg_migrations tables.  This function is called at the end of every migration, just before the transaction is finally committed.

up.erb
------
This template wraps the content of a user's migration SQL.  It creates a `SERIALIZABLE` transaction that immediately locks on `pg_migrations`, so that other clients can not happen to migrate at the same time.

Before executing the user's migration SQL, `verify_against_existing_migrations` and `bypass_existing_migration` are called to determine if the migration should proceed.  

After executing the user's migration SQL successfully, `record_migration` is called to record that the migration occured, (and won't happen again because of logic checks in bypass_existing_migration, if the migration is called again at a later time).

TODO
----
down.erb (rollbacks) and test.erb (post-migration tests)
