# DataStax C/C++ Driver for Apache Cassandra

A modern, feature-rich] and highly tunable C/C++ client library for
[Apache Cassandra] 2.1+ using exclusively Cassandra's binary protocol and
Cassandra Query Language v3. __Use the [DSE C/C++ driver] for better
compatibility and support for [DataStax Enterprise]__.

## Getting the Driver

Binary versions of the driver, available for multiple operating systems and
multiple architectures, can be obtained from our [download server]. The
source code is made available via [GitHub]. __If using [DataStax Enterprise]
use the [DSE C/C++ driver] instead__.

Packages for the driver's dependencies, libuv (1.x)
and OpenSSL, are also provided under the `dependencies` directory for each
platform (if applicable). __Note__: CentOS and Ubuntu use the version of OpenSSL
provided with the distribution:

* [CentOS 6][centos-6-dependencies]
* [CentOS 7][centos-7-dependencies]
* [Ubuntu 14.04][ubuntu-14-04-dependencies]
* [Ubuntu 16.04][ubuntu-16-04-dependencies]
* [Ubuntu 18.04][ubuntu-18-04-dependencies]
* [Windows][windows-dependencies]

## Features

* [Asynchronous API]
* [Simple], [Prepared], and [Batch] statements
* [Asynchronous I/O], [parallel execution], and request pipelining
* Connection pooling
* Automatic node discovery
* Automatic reconnection
* Configurable [load balancing]
* Works with any cluster size
* [Authentication]
* [SSL]
* [Latency-aware routing]
* [Performance metrics]
* [Tuples] and [UDTs]
* [Nested collections]
* [Retry policies]
* [Client-side timestamps]
* [Data types]
* [Idle connection heartbeats]
* Support for materialized view and secondary index metadata
* Support for clustering key order, `frozen<>` and Cassandra version metadata
* [Blacklist], [whitelist DC], and [blacklist DC] load balancing policies
* [Custom] authenticators
* [Reverse DNS] with SSL peer identity verification support
* Randomized contact points
* [Speculative execution]

## Compatibility

This driver works exclusively with the Cassandra Query Language v3 (CQL3) and
Cassandra's native protocol. The current version works with:

* Apache Cassandra versions 2.1, 2.2 and 3.0+
* Architectures: 32-bit (x86) and 64-bit (x64)
* Compilers: GCC 4.1.2+, Clang 3.4+, and MSVC 2010/2012/2013/2015/2017

If using [DataStax Enterprise] the [DSE C/C++ driver] provides more features and
better compatibility. A complete compatibility matrix for both Apache Cassandra
and DataStax Enterprise can be found [here][cpp-driver-compatability-matrix].

__Disclaimer__: DataStax products do not support big-endian systems.

## Documentation

* [Home]
* [API]
* [Getting Started]
* [Building]

## Getting Help

* JIRA: https://datastax-oss.atlassian.net/browse/CPP
* Mailing List: https://groups.google.com/a/lists.datastax.com/forum/#!forum/cpp-driver-user
* DataStax Academy via Slack: https://academy.datastax.com/slack

## Feedback Requested

**Help us focus our efforts!** [Provide your input] on the C/C++ Driver Platform
and Runtime Survey (we kept it short).

## Examples

The driver includes several examples in the [examples] directory.

## A Simple Example
```c
#include <cassandra.h>
#include <stdio.h>

int main(int argc, char* argv[]) {
  /* Setup and connect to cluster */
  CassFuture* connect_future = NULL;
  CassCluster* cluster = cass_cluster_new();
  CassSession* session = cass_session_new();
  char* hosts = "127.0.0.1";
  if (argc > 1) {
    hosts = argv[1];
  }

  /* Add contact points */
  cass_cluster_set_contact_points(cluster, hosts);

  /* Provide the cluster object as configuration to connect the session */
  connect_future = cass_session_connect(session, cluster);

  if (cass_future_error_code(connect_future) == CASS_OK) {
    CassFuture* close_future = NULL;

    /* Build statement and execute query */
    const char* query = "SELECT release_version FROM system.local";
    CassStatement* statement = cass_statement_new(query, 0);

    CassFuture* result_future = cass_session_execute(session, statement);

    if (cass_future_error_code(result_future) == CASS_OK) {
      /* Retrieve result set and get the first row */
      const CassResult* result = cass_future_get_result(result_future);
      const CassRow* row = cass_result_first_row(result);

      if (row) {
        const CassValue* value = cass_row_get_column_by_name(row, "release_version");

        const char* release_version;
        size_t release_version_length;
        cass_value_get_string(value, &release_version, &release_version_length);
        printf("release_version: '%.*s'\n", (int)release_version_length, release_version);
      }

      cass_result_free(result);
    } else {
      /* Handle error */
      const char* message;
      size_t message_length;
      cass_future_error_message(result_future, &message, &message_length);
      fprintf(stderr, "Unable to run query: '%.*s'\n", (int)message_length, message);
    }

    cass_statement_free(statement);
    cass_future_free(result_future);

    /* Close the session */
    close_future = cass_session_close(session);
    cass_future_wait(close_future);
    cass_future_free(close_future);
  } else {
    /* Handle error */
    const char* message;
    size_t message_length;
    cass_future_error_message(connect_future, &message, &message_length);
    fprintf(stderr, "Unable to connect: '%.*s'\n", (int)message_length, message);
  }

  cass_future_free(connect_future);
  cass_cluster_free(cluster);
  cass_session_free(session);

  return 0;
}
```

