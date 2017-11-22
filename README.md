# ripe-atlas

* RIPE Atlas v2 API access in Go. *

[![godoc](https://img.shields.io/badge/godoc-reference-blue.svg?style=flat)](https://godoc.org/github.com/keltia/ripe-atlas) [![license](https://img.shields.io/badge/license-MIT-red.svg?style=flat)](https://raw.githubusercontent.com/keltia/ripe-atlas/master/LICENSE) [![build](https://img.shields.io/travis/keltia/ripe-atlas.svg?style=flat)](https://travis-ci.org/keltia/ripe-atlas) [![Go Report Card](https://goreportcard.com/badge/github.com/keltia/ripe-atlas)](https://goreportcard.com/report/github.com/keltia/ripe-atlas)

`ripe-atlas` is a [Go](https://golang.org/) library to access the RIPE Atlas [REST API](https://atlas.ripe.net/docs/api/v2/manual/).

It features a simple CLI-based tool called `atlas` which serve both as a collection of use-cases for the library and an easy way to use it.

**Work in progress, still incomplete**

## Table of content
 
- [Features](#features)
- [Installation](#installation)
- [API usage](#api-usage)
  - [Basics](#basics)
- [CLI Utility](#cli-utility)
  - [Configuration](#configuration)
  - [Proxy Authentication](#proxy-authentication)
  - [Usage](#usage) 
- [TODO](#todo)
- [External Documentation](#external-documentation)
- [Contributing](#contributing)

## Features

I am trying to implement the full REST API in Go.  The API itself is not particularly complex but the settings and parameters are.

The following topic are available:

- probes

  you can query one probe or ask for a list of probes with a few criterias
  
- measurements

  you can create and list measurements.
  
- results

  every measurement has a URI in the result json that points to the actual results. This fetch and display them. 

In addition to these major commands, there are a few shortcut commands (see below):

- dns
- http
- ip
- keys
- ntp
- ping
- sslcert/tls
- traceroute

## Installation

NOTE: you MUST have Go 1.8 or later.  Previous versions did not have the `ProxyHeader` fields and thus no support for HTTP proxy.

Like many Go-based tools, installation is very easy
  
    go get github.com/keltia/ripe-atlas/cmd/...

or
  
    git clone https://github.com/keltia/ripe-atlas
    make install
    
The library is fetched, compiled and installed in whichever directory is specified by `$GOPATH`.  The `atlas` binary will also be installed (on windows, this will be called `atlas.exe`). 

You can install the dependencies with `go get`
  
- `github.com/urfave/cli`
- `github.com/naoina/toml`

To run the tests, you will also need:

- `github.com/stretchr/assert`

NOTE: please use and test the Windows version (use `make windows`to generate it).  It should work but I lack resources to play much with it.

## API usage

You must foremost instanciate a new API client with

    client, err := atlas.NewClient(config)

where `config` is an `atlas.Config{}` struct with various options.

All API calls after that will use `client`:

    probe, err := client.GetProbe(12345)

### Basics

- Authentication
- Probes
- Measurements
- Applications

## CLI utility

The `atlas` command is a command-line client for the Go API.

### Configuration

The `atlas` utility uses a configuration file in the [TOML](https://github.com/naoina/toml) file format.

On UNIX, it is located in `$HOME/.config/ripe-atlas/config.toml` and in `%LOCALAPPDATA%\RIPE-ATLAS` on Windows. 

There are only a few parameters for now, the most important one being your API Key for authenticate against the RIPE API endpoint.  You can now specify the default probe set (and override it from the CLI):

    # Default configuration file
    
    API_key = "<INSERT-API-KEY>"
    default_probe = <PROBE-ID>
    
    [probe_set]
    
    pool_size = <POOL-SIZE>
    type = "area"
    value = "WW"

Everything is a string except for `pool_size` and `default_probe` which are integers. 

Be aware that if you ask for an IPv6 object (like a domain or machine name), the API will refuse your request if the IPv6 version of that object does not exist.

### Important note

Not all parameters specified for the different commands are implemented, as you can see in the [API Reference](https://atlas.ripe.net/docs/api/v2/reference/), there are *a lot* of different parameters like all the `id__{gt,gte,lt,lte,in}` stuff.

### Proxy authentication

If you want to use `atlas` or the API behind an authenticating HTTP/HTTPS proxy, you need to create another configuration file holding the proxy credentials.  The file is called `.dbrc` under UNIX and is located in your `$HOME` directory.  On Windows, it is in the same directory as `config.toml` inside `%LOCALAPPDATA%\RIPE-ATLAS`.

The format is inherited from a Perl package I've used before.  You have a series of line for each service and contains both username and password:

    <service>   <username>  <password>      <tag>

in our case, the `service` is `proxy`, you fill username and password and `tag` will be ignored.

    proxy   john.doe    secret      ignored

For the API, you will to generate the  `username:password` string, encode it in base64 and give it to `atlas.NewClient()` with the `ProxyAuth` parameter (see `config.go` for `setupProxyAuth()`).
    
    atlas.go:
    
    // Check whether we have proxy authentication (from a separate config file)
    auth, err := setupProxyAuth()

    client, err = atlas.NewClient(atlas.Config{
        APIKey:       mycnf.APIKey,
        DefaultProbe: mycnf.DefaultProbe,
        PoolSize:     mycnf.PoolSize,
        ProxyAuth:    auth,
        Verbose:      fVerbose,
    })

As an alternative, you can do the encoding yourself and put that in `config.toml` as `proxy_auth`.    

### Usage

```
NAME:
   atlas - RIPE Atlas CLI interface

USAGE:
   atlas [global options] command [command options] [arguments...]

VERSION:
   0.23

AUTHOR(S):
   Ollivier Robert <roberto@keltia.net>

COMMANDS:
     credits, c                 credits-related keywords
     dns, dig, drill            send dns queries
     http, https                connect to host/IP through HTTP
     ip                         returns current ip
     keys, k, key               key-related keywords
     measurements, measures, m  measurements-related keywords
     ntp                        get time from ntp server
     ping                       ping selected address
     probes, p, pb              probe-related keywords
     results, r, res            results for one measurement
     sslcert, tlscert, tls      get TLS certificate from host/IP
     traceroute, trace          traceroute to given host/IP
     help, h                    Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --format value, -f value      specify output format (NOT IMPLEMENTED)
   --debug, -D                   debug mode
   --verbose, -v                 verbose mode
   --fields value, -F value      specify which fields are wanted
   --include value, -I value     specify whether objects should be expanded
   --logfile value, -L value     specify a log file
   --mine, -M                    limit output to my objects
   --opt-fields value, -O value  specify which optional fields are wanted
   --page-size value, -P value   page size for results
   --sort value, -S value        sort results
   -1, --is-oneoff               one-time measurement
   -6, --ipv6                    Only IPv6
   -4, --ipv4                    Only IPv4
   --pool-size value             Number of probes to request (default: 0)
   --area-type value             Set type for probes (area, country, etc.)
   --area-value value            Value for the probe set (WW, West, etc.)
   --help, -h                    show help
   --version, -V
```
  
In addition to the main `probes` and `measurements` commands, it features fast-access to common tasks like `ping`and `traceroute`.

When looking at measurement results, it is very easy to use something like [jq](https://stedolan.github.io/jq) to properly display JSON data:

    atlas results <ID> | jq .

You can also analyze the results, as explained [here](https://labs.ripe.net/Members/stephane_bortzmeyer/processing-ripe-atlas-results-with-jq).

Here,to find the maximum RTT:


    % ./atlas measurements results 10185594 | jq 'map(.result[0].rtt) | max'
    24.10811

And with this jq file, to get more information from a measurement:

```
% cat ping-report.jq
map(.result) | flatten(1) | map(.rtt) | length as $total | 
 "Median: " + (sort |
      if length % 2 == 0 then .[length/2] else .[(length-1)/2] end | tostring),
 "Average: " + (map(select(. != null)) | add/length | tostring) + " ms",
 "Min: " + (map(select(. != null)) | min | tostring) + " ms",
 "Max: " + (max | tostring) + " ms",
 "Failures: " + (map(select(. == null)) | (length*100/$total) | tostring) + " %"
```


    %./atlas measurements results 10185594 |  jq --raw-output --from-file ping-report.jq
    Median: 15.068505
    Average: 15.480822916666666 ms
    Min: 3.786365 ms
    Max: 24.164375 ms
    Failures: 14.285714285714286 %

## TODO

- implement "anchors"
- more tests (and better ones!)
- better display of results
- refactoring to reduce code duplication: done
- even more tests

## External Documentation

  - [Main RIPE Atlas site](https://atlas.ripe.net/)
  - [REST API Documentation](https://atlas.ripe.net/docs/api/v2/manual/)
  - [REST API Reference](https://atlas.ripe.net/docs/api/v2/reference/)

## Contributing

Please see CONTRIBUTING.md for some simple rules.
