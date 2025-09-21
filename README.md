# gator

gator is a small CLI and web app for managing RSS/Atom feeds and posts.

**Prerequisites**

- Go (1.20+ recommended) installed: https://go.dev/doc/install
- PostgreSQL installed and running: https://www.postgresql.org/download/

**Install the CLI**
Install the `gator` CLI with `go install` from the repo root (module path is read from `go.mod`):

```bash
# from the repository root
go install
```

If your `GOPATH/bin` (or the new `GOBIN`) is in your PATH, the `gator` binary will be available as `gator`.

**Configuration**
gator looks for a config file in your home directory named `.gatorconfig.json` by default. Create it with the following fields:

```json
{
  "db_url": "postgres://username:password@localhost:5432/gatordb?sslmode=disable"
}
```

Replace the `db_url` with your Postgres connection string.

Before running commands, apply the SQL schema migrations found in the `sql/schema` directory to your database. The files are written in a goose-style format with `-- +goose Up` and `-- +goose Down` sections. You can apply only the Up sections manually, or use a migration tool such as `goose`.

Quick manual apply (example, macOS zsh):

```bash
# set DBURL from your config
DBURL=$(jq -r .db_url ~/.gatorconfig.json)

# apply the Up section of each migration
awk '/-- +goose Up/{flag=1;next}/-- +goose Down/{flag=0}flag' sql/schema/001_users.sql | psql "$DBURL"
awk '/-- +goose Up/{flag=1;next}/-- +goose Down/{flag=0}flag' sql/schema/002_feeds.sql | psql "$DBURL"
awk '/-- +goose Up/{flag=1;next}/-- +goose Down/{flag=0}flag' sql/schema/003_feed_follows.sql | psql "$DBURL"
awk '/-- +goose Up/{flag=1;next}/-- +goose Down/{flag=0}flag' sql/schema/004_feed_lastfetched.sql | psql "$DBURL"
awk '/-- +goose Up/{flag=1;next}/-- +goose Down/{flag=0}flag' sql/schema/005_posts.sql | psql "$DBURL"
```

Or install `goose` and run `goose up` pointing at the same DB.

**Common commands**

- Add a feed: `gator addfeed "Feed Name" "https://example.com/rss"`
- List feeds: `gator listfeeds`
- Fetch posts: `gator fetch` (fetches unread/new posts)
- Run web server: `gator serve` (starts the HTTP server)

**Run directly**
You can also run commands directly without installing the binary using `go run` from the repo root, for example:

```bash
go run . addfeed "Hacker News RSS" "https://hnrss.org/newest"
```

If you hit `pq: relation "feeds" does not exist`, apply the migrations as shown above — the database tables are missing.
