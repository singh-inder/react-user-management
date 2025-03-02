![](https://supabase.com/docs/img/user-management-demo.png)

# Supabase Vite User Management

⭐ This example is sourced from [supabase/react-user-management](https://github.com/supabase/supabase/tree/master/examples/user-management/react-user-management).

This example will set you up for a very common situation: users can sign up with a magic link and then update their account with public profile information, including a profile image.

This demonstrates how to use:

- User signups using Supabase [Auth](https://supabase.com/auth).
- User avatar images using Supabase [Storage](https://supabase.com/storage).
- Public profiles restricted with [Policies](https://supabase.com/docs/guides/auth#policies).
- Frontend using [Vite](https://vitejs.dev/).

## Build from scratch

### 1. Set up a Supabase self-hosted instance:

You can use [supabase-automated-self-host](https://github.com/singh-inder/supabase-automated-self-host) to quickly set one up or use your own preferred way.

### 2. Run "User Management" Quickstart

Once your instance has started, head over to your project's `SQL Editor` and run the "User Management Starter" quickstart. On the `SQL editor` page, scroll down until you see `User Management Starter: Sets up a public Profiles table which you can access with your API`. Click that, then click `RUN` to execute that query and create a new `profiles` table. When that's finished, head over to the `Table Editor` and see your new `profiles` table.

### 3. Get the URL and Key

- URL will be the domain at which you've deployed your supabase.

- Get your ANON_KEY. If you've deployed self-hosted supabase instance using [supabase-automated-self-host](https://github.com/singh-inder/supabase-automated-self-host), then the ANON_KEY is present inside `supabase-automated-self-host/docker/.env` file. You can simply run the following cmd to filter the ANON_KEY:
  ```bash
  cat .env | grep ANON_KEY
  ```

The `anon` key is your client-side API key. It allows "anonymous access" to your database, until the user has logged in. Once they have logged in, the keys will switch to the user's own login token. This enables row level security for your data. Read more about this [below](#postgres-row-level-security).

**_NOTE_**: The `service_role` key has full access to your data, bypassing any security policies. These keys have to be kept secret and are meant to be used in server environments and never on a client or browser.

### 4. Env vars

Create a copy of `.env.example`, name it `.env.local` and populate this file with your URL and Key.

```
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
```

### 5. Run the application

Run the application: `npm run dev`. Open your browser to the url indicated in the CLI (eg `https://localhost:5173/`) and you are ready to go 🚀.

## Supabase details

### Postgres Row level security

This project uses very high-level Authorization using Postgres' Row Level Security.
When you start a Postgres database on Supabase, we populate it with an `auth` schema, and some helper functions.
When a user logs in, they are issued a JWT with the role `authenticated` and their UUID.
We can use these details to provide fine-grained control over what each user can and cannot do.

This is a trimmed-down schema, with the policies:

```sql
-- Create a table for Public Profiles
create table
  profiles (
    id uuid references auth.users not null,
    updated_at timestamp
    with
      time zone,
      username text unique,
      avatar_url text,
      website text,
      primary key (id),
      unique (username),
      constraint username_length check (char_length(username) >= 3)
  );

alter table
  profiles enable row level security;

create policy "Public profiles are viewable by everyone." on profiles for
select
  using (true);

create policy "Users can insert their own profile." on profiles for insert
with
  check ((select auth.uid()) = id);

create policy "Users can update own profile." on profiles for
update
  using ((select auth.uid()) = id);

-- Set up Realtime!
begin;

drop
  publication if exists supabase_realtime;

create publication supabase_realtime;

commit;

alter
  publication supabase_realtime add table profiles;

-- Set up Storage!
insert into
  storage.buckets (id, name)
values
  ('avatars', 'avatars');

create policy "Avatar images are publicly accessible." on storage.objects for
select
  using (bucket_id = 'avatars');

create policy "Anyone can upload an avatar." on storage.objects for insert
with
  check (bucket_id = 'avatars');
```

## Authors

- [Supabase](https://supabase.com)

Supabase is open source https://github.com/supabase/supabase
