<img align="left" src="banner.png" height="180px" />

## Goals

- Prevent user enumeration.

  All routes should be protected against user enumeration.

- Always confirm any change to security settings using two-factor method if available.
- Prevent phishing attacks.

## Play around with RAuth API

You can play around with the API by using the provided example and using Swagger:

```bash
# Clone the project
git clone https://github.com/insertish/rauth
cd rauth

# Bring up MongoDB
docker-compose up -d database

# Start the example
cargo run --example rocket_mongodb --features example
```

Now you can navigate to http://localhost:11111/swagger!

## Usage

Getting started is very simple, create a new instance of the RAuth struct and mount it on to Rocket.

```rust
TODO: update this
```

## Testing

To test the library, pull up required services:

```bash
# Start MongoDB and Sendria
docker-compose up -d
```

Then you can run the tests:

```bash
# Run cargo test
cargo test --features test

# Or using nextest
cargo --features test nextest run
```

## Database Migrations

Migrating the database is easy, you just have to orchestrate it yourself, ideally you have your own versioned migration system which you can slot changes into.

```rust
use rauth::{ Database, Migration };

// Acquire the database first
let database = Database::[..];

// Then run a specific migration
database.run_migration(Migration::[..]).await.unwrap();
```

The following migrations are available and must be run in order:

| Date       | Migration                   | Description                                                                                          |
| ---------- | --------------------------- | ---------------------------------------------------------------------------------------------------- |
| 2022-06-03 | `M2022_06_03EnsureUpToSpec` | Reset and reconstruct indexes to be fully up to date. This will also create any missing collections. |

## How does rAuth work?

rAuth uses email / password combinations to authenticate users and nothing else, this might not be what you're looking for but I personally prefer this format.

- If you need usernames, you need to handle this on your end.

When a user signs in, a new session is created, every single device a user logs in on has a unique session.

- This means a user can then log themselves out of old sessions or otherwise see where they are logged in.

![Example from Revolt App](https://img.insrt.uk/xexu7/daLinuSa38.png/raw)

Internally rAuth stores emails with and without special characters, `+.`.

- This means we can support plus signing without allowing the same email to sign up multiple times.
  - For example, `inbox+a@example.com` and `inbox+b@example.com` are treated as equal.
  - But since we are still storing the original email, we still send them marked with the user's sign.
- In the case of Gmail, all emails with dots are forwarded to those without them, this can lead to some [unfortunate situations](https://jameshfisher.com/2018/04/07/the-dots-do-matter-how-to-scam-a-gmail-user/).
  - Generally, we treat all emails with dots as their non-dot counterpart when checking if an email exists.
  - This may inconvenience some users but I would rather avoid situations like above or duplicate accounts.
- When logging in, we use the normalised email to find the correct account.
