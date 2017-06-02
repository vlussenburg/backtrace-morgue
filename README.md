# morgue

## Installation

It is recommended to install `morgue` using `npm`.

```
npm install backtrace-morgue -g
```

If you working from the repository, then instead use the following command.
```
npm install -g
```

This will install the `morgue` tool in your configured path. Refer to the
`morgue --help` command to learn more.

## Introduction

`morgue` is a command-line interface to the Backtrace object store. It allows
you to upload, download and issue queries on objects with-in the object store.

## Usage

### login

```
Usage: morgue login <url>
```

The first step to using `morgue` is to log into a server.

```
$ morgue login http://localhost
User: sbahra
Password: **************

Logged in.
```

At this point, you are able to issue queries.

### describe

```
Usage: morgue describe <[<universe>/]project> [substring]
```

Requests a list and description of all metadata that can be queried against.

#### Example

```
$ morgue describe bidder uname
              uname.machine: machine hardware name
              uname.release: kernel release
              uname.sysname: kernel name
              uname.version: kernel version
```

### get

```
Usage: morgue get <[<universe>/]project> <object id> [-o <output file>]
```

Downloads the specified object from the Backtrace object store and prints
to standard output. Optionally, output the file to disk.

### put

```
Usage: morgue put <[<universe>/]project> <file> <--format=btt|minidump|json|symbols> [options]
```

Uploads object file to the Backtrace object store. User has the following options

| Option        | Description |
|---------------|-------------|
| `--compression=gzip|deflate`| uploaded file is compressed |
| `--kv=key1:value1,key2:value2,...`| upload key-values |
| `--form_data`| upload file by multipart/form-data post request |

### modify

```
Usage: morgue modify <[universe/]project> <object id> [--set ...] [--clear ...]

Modifies attributes of the given object in the manner specified.
Both options below may be specified more than once.

| Option        | Description |
|---------------|-------------|
| `--set`       | Set the given `attribute=value` pair |
| `--clear`     | Clear the given `attribute` |
```

#### Example

```
$ morgue modify --set hostname=fqdn.example.com myproject 0
```

### list

Allows you to perform queries on object metadata. You can perform
either selection queries or aggregation queries, but not both at the
same time.

```
Usage: morgue list <[<universe>/]project> [substring]
```

#### Filters

The filter option expects a comma-delimited list of the form
`<attribute>,<operation>,<value>`.

The currently supported operations are `equal`, `regular-expression`,
`at-least`, `greater-than`, `at-most` and `less-than`.

#### Aggregations

Aggregation is expressed through a myriad of command-line options that express
different aggregation operations. Options are of form `--<option>=<attribute>`.

The ``*`` factor is used when aggregations are performed when no factor is
specified or if an object does not have a valid value associated with the
factor.

| Option        | Description |
|---------------|-------------|
| `--unique`    | provide a count of distinct values |
| `--histogram` | provide all distinct values |
| `--sum`       | sum all values |
| `--range`     | provide the minimum and maximum values |
| `--count`     | count all non-null values |
| `--bin`       | provide a linear histogram of values |
| `--head`      | provide the first value in a factor |
| `--tail`      | provide the last value in a factor |
| `--object`    | provide the maximum object identifier of a column |

#### Example

Request all faults from application deployments owned by jdoe.
Provide the timestamp, hostname, callstack and classifiers.

```
$ morgue list bidder --filter=tag_owner,equal,jdoe --select=timestamp --select=hostname --select=callstack --select=classifiers
*                               
#9d33    Thu Oct 13 2016 18:36:01 GMT-0400 (EDT)     5 months ago
  hostname: 2235.bm-bidderc.prod.nym2
  classifiers: abort stop
  callstack:
    assert ← int_set_union_all ← all_domain_lists ← 
    setup_phase_unlocked ← bid_handler_slave_inner ← bid_handler_slave ← 
    an_sched_process_task ← an_sched_slave ← event_base_loop ← 
    an_sched_enter ← bidder_slave ← an_sched_pthread_cb
#ef2f    Thu Oct 13 2016 18:36:01 GMT-0400 (EDT)     5 months ago
  hostname: 2066.bm-impbus.prod.nym2
  classifiers: abort stop
  callstack:
    assert ← an_discovery_get_instances ← budget_init_discovery ← 
    main
#119bf   Thu Oct 13 2016 18:36:01 GMT-0400 (EDT)     5 months ago
  hostname: 2066.bm-impbus.prod.nym2
  classifiers: abort stop
  callstack:
    assert ← an_discovery_get_instances ← budget_init_discovery ← 
    main
```

Request faults owned by jdoe, group them by fingerprint and aggregate
the number of unique hosts, display a histogram of affected versions and
provide a linear histogram of process age distributon.

```
$ morgue list bidder --age=1y --factor=fingerprint --filter=tag_owner,equal,jdoe --head=callstack --unique=hostname --histogram=tag --bin=process.age
823a55fb15bf697ba3041d736ade... ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁█▁▁▁▁▁▁▁▁▁▁▁▁ 5 months ago
Date: Wed May 18 2016 18:44:35 GMT-0400 (EDT)
callstack:
    assert ← int_set_union_all ← all_domain_lists ← 
    setup_phase_unlocked ← bid_handler_slave_inner ← bid_handler_slave ← 
    an_sched_process_task ← an_sched_slave ← event_base_loop ← 
    an_sched_enter ← bidder_slave ← an_sched_pthread_cb
histogram(tag): 
  8.20.4.adc783.0 ▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆ 1
unique(hostname): 1
bin(process.age): 
          7731         7732 ▆▆▆▆▆▆▆▆▆▆ 1

3b851ac1ab1421409159cc38edb2... ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁█▁▁▁▁▁▁▁▁▁▁▁▁▁ 5 months ago
Date: Tue May 17 2016 17:28:26 GMT-0400 (EDT)
      Tue May 17 2016 17:30:07 GMT-0400 (EDT)
callstack:
    assert ← an_discovery_get_instances ← budget_init_discovery ← 
    main
histogram(tag): 
  4.44.0.adc783.1 ▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆ 2
unique(hostname): 1
bin(process.age): 
            23           24 ▆▆▆▆▆▆▆▆▆▆ 1
            24           25 ▆▆▆▆▆▆▆▆▆▆ 1
```

### delete

Allows deleting objects.

```
Usage: morgue delete <[universe/]project> <oid1> [... oidN]
```

Object IDs must be specified; they can be found in `morgue list` output.
The object ID printed in the example above is `9d33`.

### flamegraph

```
Usage: morgue flamegraph [--filter=<filter expression>] [--reverse] [--unique] [-o file.svg]
```

Generate a flamegraph of callstacks of all objects matching the specified
filter criteria. The `--filter` option behaves identically to the `list`
sub-command. This functionality requires `perl` to be installed.
To learn more about flamegraphs, please see
http://www.brendangregg.com/flamegraphs.html.

Use `--unique` to only sample unique crashes. Use `--reverse` to begin sampling
from leaf functions.

### report

```
Usage: morgue report <project> [-l <layout.json>] [-o <report.html>]
```

The morgue report command generates a weekly digest of activity on a server.
The report is output as a stand-alone HTML file that is suitable for e-mail.
If the `-o` option is not specified, the default filename is `report.html`.
Reports can be tuned with the help of a JSON specification file passed to the
`-l` option. An example specification file is below, the `_` keys represent
comments.

```
{

  "_" : "Crashes by top attributes over reporting period",
  "_" : "Multiple queries can be specified with different filters"
  "top" : [
    {
      "title" : null,
      "attributes" : ["application", "uname.sysname"],
      "filter" : []
    }
  ],

  "feed" : {
    "_" : "The number of unique groups to show",
    "limit" : 5
  }
}
```

### symbol

```
Usage: morgue symbol <[<universe>/]project> [status | list] [-o <output file>]
```

Retrieve a list of uploaded symbols or symbol archives. By default, `morgue symbol`
will return a list of uploaded, in-process and symbol processing errors. If `list`
is used, then a list of uploaded symbols is returned.

### setup

```
Usage: morgue setup <url>
```

If you are using an on-premise version of `coronerd`, use `morgue setup`
to configure the initial organization and user. For example, if the server is
`backtrace.mycompany.com`, then you would run `morgue setup http://backtrace.mycompany.com`.
We recommend resetting your password after you enable SSL (done by configuring
your certificates).


