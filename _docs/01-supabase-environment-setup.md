```sql
DROP POLICY
  "Avatar images are publicly accessible." ON storage.objects;

DROP POLICY
  "Anyone can upload an avatar." ON storage.objects;

DELETE FROM
  storage.buckets
WHERE
  id = 'avatars';

drop trigger
  on_auth_user_created on auth.users;

DROP FUNCTION
  public.handle_new_user();

DROP TABLE
  profiles;

-- Create a table for public profiles
create table
  profiles (
    id uuid references auth.users on delete cascade not null primary key,
    updated_at timestamp with time zone,
    username text unique,
    full_name text,
    avatar_url text,
    website text,
    email text,
    constraint username_length check (char_length(username) >= 3)
  );

-- Set up Row Level Security (RLS)
-- See https://supabase.com/docs/guides/auth/row-level-security for more details.
alter table
  profiles enable row level security;

create policy
  "Public profiles are viewable by everyone." on profiles for
select
  using (true);

create policy
  "Users can insert their own profile." on profiles for insert
with
  check (auth.uid () = id);

create policy
  "Users can update own profile." on profiles for
update
  using (auth.uid () = id);

-- This trigger automatically creates a profile entry when a new user signs up via Supabase Auth.
-- See https://supabase.com/docs/guides/auth/managing-user-data#using-triggers for more details.
create function
  public.handle_new_user () returns trigger as $$
begin
  insert into public.profiles (id, full_name, avatar_url, email)
  values (new.id, new.raw_user_meta_data->>'full_name', new.raw_user_meta_data->>'avatar_url', new.raw_user_meta_data->>'email');
  return new;
end;
$$ language plpgsql security definer;

create trigger
  on_auth_user_created
after
  insert on auth.users for each row
execute
  procedure public.handle_new_user ();

-- Set up Storage!
insert into
  storage.buckets (id, name)
values
  ('avatars', 'avatars');

-- Set up access controls for storage.
-- See https://supabase.com/docs/guides/storage#policy-examples for more details.
create policy
  "Avatar images are publicly accessible." on storage.objects for
select
  using (bucket_id = 'avatars');

create policy
  "Anyone can upload an avatar." on storage.objects for insert
with
  check (bucket_id = 'avatars');

```

Following the docs to create a local environment and linking it to a staging environment.

The first step is to set up your local repository with the Supabase CLI:

```
supabase init
```

You should see a new supabase directory. Then you need to link your local repository with your Supabase project:

```
supabase login
supabase link --project-ref $PROJECT_ID
```

You can get your $PROJECT_ID from your project's dashboard URL:

```
https://supabase.com/dashboard/project/<project-id>
```

If you're using an existing Supabase project, you might have made schema changes through the Dashboard.
Run the following command to pull these changes before making local schema changes from the CLI:

```
supabase db pull
```

This command creates a new migration in supabase/migrations/<timestamp>\_remote_schema.sql which reflects the schema changes you have made previously.

You will also need to create a type using the supabase CLI. By using the `--local` flag we can create types based on our local db perfect for working on a local environment.

```json
"scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    // Add this line to your scripts
    "update-types": "npx supabase gen types typescript --local > ./src/lib/types/database.types.ts"
  },
```

Now commit your local changes to Git and run the local development setup:

```
git add .
git commit -m "init supabase"
supabase start
```

You are now ready to develop schema changes locally and create your first migration.
