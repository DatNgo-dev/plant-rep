By running `npx supabase init` we can create a local supabase db server with the dashboard UI. In the dashboard UI we can create migrations for updating tables, triggers, abilities and so on.

They are two ways to create migration in supabase CLI:

1. Manual
2. Auto Schema diff

## Manual

We can write DDL statements manually into a migration file.

- Create a new migration script by running: `npx supabase migration new new_employee`
- You should see a new file created: supabase/migrations/<timestamp>\_new_employee.sql. You can then write SQL statements in this script using a text editor:

```sql
create table public.employees (
  id integer primary key generated always as identity,
  name text
);
```

- Apply the new migration to your local database: `supabase db reset`

This command recreates your local database from scratch and applies all migration scripts under supabase/migrations directory. Now your local database is up to date.

The new migration command also supports stdin as input.
This allows you to pipe in an existing script from another file or stdout:

`npx supabase migration new new_employee < create_employees_table.sql`

## Auto schema diff

Unlike manual migrations, auto schema diff creates a new migration script from changes already applied to your local database.

Create an employees table under the public schema using Studio UI, accessible at localhost:54323 by default.

Next, generate a schema diff by running the following command:

`npx supabase db diff -f name_of_migration`

or

`npx supabase db diff --use-migra name_of_migration`

\*you may pass in the --use-migra experimental flag to generate a more concise migration using migra.
Without the -f file flag, the output is written to stdout by default.

migration name example: add_gender_to_profiles
You should see that a new file supabase/migrations/<timestamp>\_new_employee.sql is created.

After creating a migration using either method, commit your changes and get ready to deploy!
