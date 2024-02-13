# Environment variables

Main goals:

- different variables for different binary
- allow secrets to be but in a file, instead of an environment variable or a command line parameter
- command line parameters should override environment variables
- there's no need for a public key (file or environment variable): this can be derived from the secret key
- each binary allow environment to be automatically loaded (and overridden) from an `.env` file
- hbbr/hbbs should be able to run from ENV only (plus DB for hbbs), cli args only, `.env` file only, or a mix

## hbbs

| variable                 | previous                 | default value | cli arg              | meaning                                                               |
| ------------------------ | ------------------------ | ------------- | -------------------- | --------------------------------------------------------------------- |
| HBBS_SECRET_KEY          | KEY                      | -             | --key                | the secret key                                                        |
| HBBS_SECRET_KEY_FILE     | -                        | -             |                      | path to a file containing the secret key                              |
| HBBS_DB_FILE             | DB_URL                   | db_v2.sqlite3 |                      | path to the sqlite3 database file                                     |
| HBBS_ALLOW_UNENCRYPTED   | -                        | N             |                      | if "Y" explicitly allow non encrypted connections                     |
| HBBS_ALLOW_ANY_KEY       | -                        | N             |                      | if "Y" explicitly allow any encryption key                            |
| HBBS_ALWAYS_USE_RELAY    | ALWAYS_USE_RELAY         | N             |                      | If "Y" disallow direct peer connection                                |
| HBBS_RELAY_SERVERS       | RELAY_SERVERS            | -             | --relay-servers      | IP address/DNS name of the machines running hbbr (separated by comma) |
| HBBS_PORT                | PORT                     | 21116         | --port               | hbbs listening port                                                   |
| MAX_DATABASE_CONNECTIONS | MAX_DATABASE_CONNECTIONS | 1             |                      |                                                                       |
| RUST_LOG                 | RUST_LOG                 | error         |                      | set debug level (error, warn, info, debug, trace)                     |
|                          |                          |               | --config             | custom config file                                                    |
|                          |                          |               | --serial             | update serial number                                                  |
|                          |                          |               | --software-url       | new release download url                                              |
|                          |                          |               | --rmem               | Sets UDP recv buffer size                                             |
|                          |                          |               | --local-ip           |                                                                       |
|                          |                          |               | --mask               | Determine if the connection comes from LAN, e.g. 192.168.0.0/16       |
|                          |                          |               | --rendezvous_servers | ??                                                                    |

Notes:

- `HBBS_SECRET_KEY` and `HBBS_SECRET_KEY_FILE` are mutually exclusive. The program will stop for configuration error if both are specified.
- `HBBS_SECRET_KEY` (or `HBBS_SECRET_KEY_FILE`) is required unless `HBBS_ALLOW_UNENCRYPTED=1` or `HBBS_ALLOW_ANY_KEY=1` is specified
- `HBBS_SECRET_KEY` should be a valid key (`-` is not allowed, `-` is not allowed)
- `HBBS_SECRET_KEY_FILE` should refer to a file containing a valid key (`-` is not allowed, `-` is not allowed)
- `KEY` should be a valid key or `-` or `-`
- `HBBS_DB_FILE` is required. The destination file is created if not already present.

## hbbr

| variable                   | previous              | default value | cli arg | meaning                                                |
| -------------------------- | --------------------- | ------------- | ------- | ------------------------------------------------------ |
| HBBR_SECRET_KEY            | KEY                   | -             | --key   | the secret key                                         |
| HBBR_SECRET_KEY_FILE       | -                     | -             |         | path to a file containing the secret key               |
| HBBR_ALLOW_UNENCRYPTED     | -                     | N             |         | if "Y" explicitly allow non encrypted connections      |
| HBBR_ALLOW_ANY_KEY         | -                     | N             |         | if "Y" explicitly allow any encryption key             |
| HBBR_PORT                  | PORT                  | 21117         | --port  | hbbr listening port                                    |
| HBBR_DOWNGRADE_START_CHECK | DOWNGRADE_START_CHECK | 1800          |         | delay (in seconds) before downgrade check              |
| HBBR_DOWNGRADE_THRESHOLD   | DOWNGRADE_THRESHOLD   | 0.66          |         | threshold of downgrade check (bit/ms)                  |
| HBBR_LIMIT_SPEED           | LIMIT_SPEED           | 4             |         | speed limit (in Mb/s)                                  |
| HBBR_SINGLE_BANDWIDTH      | SINGLE_BANDWIDTH      | 16            |         | max bandwidth for a single connection (in Mb/s)        |
| HBBR_TOTAL_BANDWIDTH       | TOTAL_BANDWIDTH       | 1024          |         | max total bandwidth (in Mb/s)                          |
| HBBR_BLACKLIST_FILE        |                       | blacklist.txt |         | path to a file containing a blacklist                  |
| HBBR_BLOCKLIST_FILE        |                       | blocklist.txt |         | path to a file containing a blocklist                  |
| RUST_LOG                   | RUST_LOG              | error         |         | set debug level (off, error, warn, info, debug, trace) |

Notes:

- `HBBR_SECRET_KEY` and `HBBR_SECRET_KEY_FILE` are mutually exclusive. The program will stop for configuration error if both are specified.
- `HBBR_SECRET_KEY` (or `HBBR_SECRET_KEY_FILE`) is required unless `HBBR_ALLOW_UNENCRYPTED=1` or `HBBR_ALLOW_ANY_KEY=1` is specified
- `HBBR_SECRET_KEY` should be a valid key (`-` is not allowed, `-` is not allowed)
- `HBBR_SECRET_KEY_FILE` should refer to a file containing a valid key (`-` is not allowed, `-` is not allowed)
- `KEY` should be a valid key or `_` or `–`

## error logging

It is possible to specify different logging level for both programs, by setting `RUST_LOG` in this way: `RUST_LOG=hbbr=warn,hbbs=off`.

## some clarification needed

- (hbbr) What is the difference between blacklist.txt and blocklist.txt? And why is this not used in hbbs?
- (hbbs) What is the meaning of the environment variable `TEST_HBBS`? There's only a comment saying "temp solution to solve udp socket failure". I understand that this can be an IP address (maybe the external IP of the server when natted), but it keeps failing for me with "Error: invalid socket address syntax"
- (hbbs) What is the meaning of the command line argument "--software-url"? I understand that if I set like "--software-url=http://somedomain.com/something-44", then the calculated version is "44", but where is this information used?
- (hbbs) There are some other command line arguments that are unclear (to me): `--local-ip`, `--mask`.
- (hbbs) What is the meaning of the environment variables `PORT_FOR_API` and `KEY_FOR_API`?
- (hbbs) Is `MAX_DATABASE_CONNECTIONS` needed?
- (hbbs) What is the meaning of `--rendezvous-servers`?

## other ideas

Previous environment variables should generate a deprecation warning message in the log.
It's also a good idea to indicate in which version the varibale will be removed (next major release?).

There's no specific log configuration, the only option is setting the `RUST_LOG` variable; this has to be improved. One idea is to differentiate log levels for the two binaries, another is to add an option to log to a file; this should cover the vast majority of cases.

Instead of using `blacklist.txt` and `blocklist.txt`, we can use a database. Maybe the same database we use for hbbs.

Is the `.env` file really necessary? It can create confusion with docker-compose.
