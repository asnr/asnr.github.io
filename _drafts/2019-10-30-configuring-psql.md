# Configuring psql

There are a lot of Postgres databases in my workplace. Keeping track of how to connect to all of them can be a chore; this is how I do it.

The `psql` client has several defualt file locations it checks for connection strings. I use `~/.pg_service.conf` to store non-sensitive connection values and `~/.pgpass` for sensitive connection values (i.e. passwords).

For example, say there is an app called "Dough Kneader" that uses postgres for its data store. It has several environments, including a "dev" environment and a "staging" environment. In `~/.pg_service.conf` I would put

```
[dk-dev]
host=dough-kneader-dev.robot-bakers.com
port=5432
user=asnr
dbname=doughkneader

[dk-staging]
host=dough-kneader-staging.robot-bakers.com
port=5432
user=asnr
dbname=doughkneader
```

and in `~/.pgpass` I would put

```
dough-kneader-dev.robot-bakers.com:5432:doughkneader:asnr:SECRET_DEV_PASSWORD
dough-kneader-staging.robot-bakers.com:5432:doughkneader:asnr:SECRET_STAGING_PASSWORD
```

Note that the order of the fields in `~/.pgpass` is `HOST:PORT:DB:USER:PASSWORD`.

With this setup, I connect to the databases by running

```
$ psql service=dk-dev
# or
$ psql service=dk-staging
```

As I add more and more database configurations, it can be hard to remember the names I give them, so I have a simple alias that lists them all:

```sh
alias psql_hosts='grep "^\[" ~/.pg_service.conf | tr -d []'
```

If I have more than one work computer, I'll want to keep these configurations in sync between them. To do this, I'll put the `~/.pg_service.conf` file in my [dotfiles](https://dotfiles.github.io) repository. (Don't put `~/.pgpass` with all its passwords in there though! That's a good way to have a bad time.)

And that's it! Unless I need to connect to some of the databases through a SQL proxy, in which case there's a little more work I need to do.


## SQL proxies

I'm going to focus on GCP's `cloud_sql_proxy`. Connecting to a CloudSQL instance through the proxy involves starting the proxy and then, while the proxy is running, starting `psql` with a connection string pointing at the proxy.

I find it convenient to let the system's init deamon take care of running the proxy in the background.

If you're using Linux, chances are that your init system is `systemd`, in which case you can specify the `cloud_sql_proxy` service by writing the following contents to the 
`~/.config/systemd/user/cloud_sql_proxy.service` file, replacing `/<ABSOLUTE PATH TO BINARY>/cloud_sql_proxy` with absolute path to the `cloud_sql_proxy` binary on your filesystem:


```
[Unit]
Description=Cloud SQL Proxy

[Service]
Type=simple
ExecStart=/<ABSOLUTE PATH TO BINARY>/cloud_sql_proxy -instances=dev-gcp-project:us-central1:bake-manager=tcp:5300,staging-gcp-project:us-central1:bake-manager=tcp:5301

[Install]
WantedBy=multi-user.target
```

You can then load the service by executing

```
$ systemctl --user enable cloud_sql_proxy
```

and run it by executing

```
$ systemctl --user start cloud_sql_proxy
```

Once it's running, you can connect to the database by pointing `psql` to `localhost:5300` or `localhost:5301` for our hypothetical dev or staging environments respectively. Of course, you shouldn't try to remember these details yourself, but rather store them in `~/.pg_service.conf` and `~/.pgpass` as I've described above.

On macOS, we can use launchd to fill a similar role. Write the following contents to the `~/Library/LaunchAgents/cloud_sql_proxy.plist` file:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
  	<key>Label</key>
  	<string>cloud_sql_proxy</string>
    <key>ProgramArguments</key>
    <array>
      <string>/<ABSOLUTE PATH TO BINARY>/cloud_sql_proxy</string>
      <string>-instances=dev-gcp-project:us-central1:bake-manager=tcp:5300,staging-gcp-project:us-central1:bake-manager=tcp:5301</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
      <key>PATH</key>
      <string>/<ABSOLUTE PATH TO SDK>/google-cloud-sdk/bin:/usr/bin:/bin:/usr/sbin:/sbin</string>
    </dict>
  </dict>
</plist>
```

As with the `systemd` service, you need to replace `/<ABSOLUTE PATH TO BINARY>/cloud_sql_proxy` with the absolute path to the proxy binary. You also need to add the Google Cloud SDK binary directory to the `$PATH` environment variable, as `cloud_sql_proxy` uses these binaries to authenticate with GCP.

Once that's all setup, you can load the service by executing

```
$ launchctl load ~/Library/LaunchAgents/cloud_sql_proxy.plist
```

and start it by running

```
$ launchctl start cloud_sql_proxy
```
