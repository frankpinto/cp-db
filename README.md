# cp-db
As developers we often need to push and pull databases around. From production to staging to test new data-sensitive features, from production to local to debug production-only bugs, etc. Currently supports one local PostgreSQL database and one other one through DATABASE_URL (I use it for production)


### Setup
First get it:

```bash
git clone https://github.com/frankpinto/cp-db.git
```

Then set it up:

```ruby
rake setup
```

Fill in .env as per the instructions, DATABASE_URL is for production, the rest of the credentials are for local


### Usage
To download the entire database dump:

```ruby
rake down
```

To load it into local:

```ruby
rake load
```

### TODO

* Specify tables to dump
* Better error messages if credentials in .env are wrong, right now it just
  spits out pg_* errors
