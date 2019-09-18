# homebrew-tap

This is Journy's Homebrew Tap, which features formulas to help pin
Postgres to 9.4 and Postgis to 2.5.

Homebrew will pull in the latest version of formulas when they are upgraded,
meaning that users can inadvertently be upgraded to Postgresql 11. The
postgresql.rb formula here ensures that 9.4.24 is installed, and the postgis.rb
formulate ensures that 2.5.3 is installed.

## Installing Postgres 9.4 and Postgis 2.5

First ensure you have upgraded to the latest homebrew:

```sh
brew update
```

It's recommended to go ahead and clear any existing Postgres and Postgis
installations:

```sh
brew uninstall postgis --force
brew uninstall postgresql --force
brew uninstall postgresql@9.4 --force
brew cleanup
brew services stop postgresql@9.4
brew services stop postgresql
```

After this, install Clover's custom brew tap:

```sh
brew tap journy/homebrew-tap
```

Then install the latest version of Postgres and unlink it. This is done first so that
we can explicitly switch to an earlier version before installing Postgis:

```sh
brew install postgresql
brew unlink postgresql
```

The previous install of postgresql runs `initdb`, which creates database structures incompatible with 9.4.24. This needs to be removed with:

```sh
rm -rf /usr/local/var/postgres
```

Now install Postgres from this tap with:

```sh
brew install journy/tap/postgresql  # yes, without the homebrew-
```

Now you will have both 9.4.24 and the latest version of Postgres installed.
Switch to 9.4.24 with:

```sh
brew switch postgresql 9.4.24
```

Postgis 2.5 can be installed with:

```sh
brew install journy/tap/postgis
```

Try running and accessing Postgres with the following:

```sh
brew services start postgresql
psql postgres  # It should show 9.4.24 as the version on the prompt
```

After running `psql postgres`, type the following in the prompt to verify your Postgis installation:

```sh
drop extension postgis;  -- This might fail if it wasn't previously installed
create extension postgis;  -- This should pass
select ST_Distance(
  ST_GeometryFromText('POINT(-118.4079 33.9434)', 4326), -- Los Angeles (LAX)
  ST_GeometryFromText('POINT(2.5559 49.0083)', 4326)     -- Paris (CDG)
);  -- This should print a row with 121.898285970107 as a value
```

If you have to set up a `postgres` user to run `rake db:setup` in your rails application you can run:

```sh
CREATE ROLE postgres LOGIN CREATEDB SUPERUSER;
```
