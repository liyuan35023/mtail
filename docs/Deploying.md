# Introduction

mtail is intended to run one per machine, and serve as monitoring glue for multiple applications running on that machine.  It runs one or more programs in a 1:1 mapping to those client applications.

## Configuration Overview

mtail is configured through commandline flags.

The `--help` flag will print a list of flags for configuring `mtail`.

(Flags may be prefixed with either `-` or `--`)

Basic flags necessary to start `mtail`:

  * `--logs` is a comma separated list of filenames to extract from, but can also be used multiple times, and each filename can be a [glob pattern](http://godoc.org/path/filepath#Match).
  * `--progs` is a directory path containing [mtail programs](Language). Programs must have the `.mtail` suffix.

mtail runs an HTTP server on port 3903, which can be changed with the `--port` flag.

# Details

## Launching mtail

```
mtail --progs /etc/mtail --logs /var/log/syslog --logs /var/log/ntp/peerstats
```

`mtail` will start to read the specified logs from their current end-of-file,
and read new updates appended to these logs as they arrive.  It will attempt to
correctly handle log files that have been rotated by renaming or symlink
changes.

## Writing the programme

Read the [Programming Guide](Programming-Guide) for instructions on how to write an `mtail` program.

## Getting the Metrics Out

### Pull based collection

Point your collection tool at `localhost:3903/json` for JSON format metrics.

Prometheus can be directed to the /metrics endpoint for Prometheus text-based format.

### Push based collection

Use the `collectd_socketpath` or `graphite_host_port` flags to enable pushing to a collectd or graphite instance.

Configure collectd on the same machine to use the unixsock plugin, and set `collectd_socketpath` to that unix socket.

```
mtail --progs /etc/mtail --logs /var/log/syslog,/var/log/rsyncd.log --collectd_socketpath=/var/run/collectd-unixsock
```

Set `graphite_host_port` to be the host:port of the carbon server.

```
mtail --progs /etc/mtail --logs /var/log/syslog,/var/log/rsyncd.log --graphite_host_port=localhost:9999
```

Likewise, set `statsd_hostport` to the host:port of the statsd server.

Additionally, the flag `metric_push_interval_seconds` can be used to configure the push frequency.  It defaults to 60, i.e. a push every minute.

## Troubleshooting

Lots of state is logged to the log file, by default in `/tmp/mtail.INFO`.  See [Troubleshooting](Troubleshooting) for more information.

N.B. Oneshot mode (the `one_shot` flag on the commandline) can be used to check
that a program is correctly reading metrics from a log, but with the following
caveats:

* Unlike normal operations, oneshot mode will read the logs from the start of
  the file to the end, then close them -- it does not continuously tail the
  file
* The metrics will be printed to standard out when the logs are finished being
  read from.
* mtail will exit after the metrics are printed out.

This mode is useful for debugging the behaviour of `mtail` programs and
possibly for permissions checking.
