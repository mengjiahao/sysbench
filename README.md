[![Latest Release][release-badge]][release-url]
[![Build Status][travis-badge]][travis-url]
[![Debian Packages][deb-badge]][deb-url]
[![RPM Packages][rpm-badge]][rpm-url]
[![Coverage Status][coveralls-badge]][coveralls-url]
[![License][license-badge]][license-url]

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [sysbench](#sysbench)
    - [Features](#features)
- [Installing from Binary Packages](#installing-from-binary-packages)
    - [Linux](#linux)
    - [macOS](#macos)
    - [Windows](#windows)
- [Building and Installing From Source](#building-and-installing-from-source)
    - [Build Requirements](#build-requirements)
        - [Windows](#windows)
        - [Debian/Ubuntu](#debianubuntu)
        - [RHEL/CentOS](#rhelcentos)
        - [Fedora](#fedora)
        - [macOS](#macos)
    - [Build and Install](#build-and-install)
- [Usage](#usage)
    - [General Syntax](#general-syntax)
    - [General Command Line Options](#general-command-line-options)
- [Versioning](#versioning)

<!-- markdown-toc end -->

# sysbench

sysbench is a scriptable multi-threaded benchmark tool based on
LuaJIT. It is most frequently used for database benchmarks, but can also
be used to create arbitrarily complex workloads that do not involve a
database server.

sysbench comes with the following bundled benchmarks:

- `oltp_*.lua`: a collection of OLTP-like database benchmarks
- `fileio`: a filesystem-level benchmark
- `cpu`: a simple CPU benchmark
- `memory`: a memory access benchmark
- `threads`: a thread-based scheduler benchmark
- `mutex`: a POSIX mutex benchmark

## Features

- extensive statistics about rate and latency is available, including
  latency percentiles and histograms;
- low overhead even with thousands of concurrent threads. sysbench is
  capable of generating and tracking hundreds of millions of events per
  second;
- new benchmarks can be easily created by implementing pre-defined hooks
  in user-provided Lua scripts;
- can be used as a general-purpose Lua interpreter as well, simply
  replace `#!/usr/bin/lua` with `#!/usr/bin/sysbench` in your script.

# Installing from Binary Packages

## Linux

The easiest way to download and install sysbench on Linux is using
binary package repositories hosted by
[packagecloud](https://packagecloud.io). The repositories are
automatically updated on each sysbench release. Currently x86_64, i386
and aarch64 binaries are available.

Multiple methods to download and install sysbench packages are available and
described at <https://packagecloud.io/akopytov/sysbench/install>.

Quick install instructions:

- Debian/Ubuntu
  ``` shell
  curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
  sudo apt -y install sysbench
  ```

- RHEL/CentOS:
  ``` shell
  curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
  sudo yum -y install sysbench
  ```

- Fedora:
  ``` shell
  curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash	
  sudo dnf -y install sysbench
  ```

## macOS

On macOS, up-to-date sysbench packages are available from Homebrew:
```shell
# Add --with-postgresql if you need PostgreSQL support
brew install sysbench
```

## Windows
As of sysbench 1.0 support for native Windows builds was dropped. It may
be re-introduced in later releases. Currently, the recommended way to
obtain sysbench on Windows is
using
[Windows Subsystem for Linux available in Windows 10](https://msdn.microsoft.com/en-us/commandline/wsl/about).

After installing WSL and getting into he bash prompt on Windows
following Debian/Ubuntu installation instructions is
sufficient. Alternatively, one can use WSL to build and install sysbench
from source, or use an older sysbench release to build a native binary.

# Building and Installing From Source

It is recommended to install sysbench from the official binary
packages as described in
[Installing from Binary Packages](#installing-from-binary-packages). Below
are instruction for cases when you want to use sysbench on an
architecture for which no binary packages are available.

## Build Requirements

### Windows
As of sysbench 1.0 support for native Windows builds was
dropped. It may be re-introduced in later versions. Currently, the
recommended way to build sysbench on Windows is using
[Windows Subsystem for Linux available in Windows 10](https://msdn.microsoft.com/en-us/commandline/wsl/about).

After installing WSL and getting into bash prompt on Windows, following
Debian/Ubuntu build instructions is sufficient. Alternatively, one can
build and use an older 0.5 release on Windows.

### Debian/Ubuntu
``` shell
    apt -y install make automake libtool pkg-config libaio-dev
    # For MySQL support
    apt -y install libmysqlclient-dev libssl-dev
    # For PostgreSQL support
    apt -y install libpq-dev
```

### RHEL/CentOS
``` shell
    yum -y install make automake libtool pkgconfig libaio-devel
    # For MySQL support, replace with mysql-devel on RHEL/CentOS 5
    yum -y install mariadb-devel openssl-devel
    # For PostgreSQL support
    yum -y install postgresql-devel
```

### Fedora
``` shell
    dnf -y install make automake libtool pkgconfig libaio-devel
    # For MySQL support
    dnf -y install mariadb-devel openssl-devel
    # For PostgreSQL support
    dnf -y install postgresql-devel
```

### macOS

Assuming you have Xcode (or Xcode Commane Line Tools) and Homebrew installed:
``` shell
    brew install automake libtool openssl pkg-config
    # For MySQL support
    brew install mysql
    # For PostgreSQL support
    brew install postgresql
    # openssl is not linked by Homebrew, this is to avoid "ld: library not found for -lssl"
    export LDFLAGS=-L/usr/local/opt/openssl/lib 
```

## Build and Install
``` shell
    ./autogen.sh
    # Add --with-pgsql to build with PostgreSQL support
    ./configure
    make -j
    make install
```

The above will build sysbench with MySQL support by default. If you have
MySQL headers and libraries in non-standard locations (and no
`mysql_config` can be found in the `PATH`), you can specify them
explicitly with `--with-mysql-includes` and `--with-mysql-libs` options
to `./configure`.

To compile sysbench without MySQL support, use `--without-mysql`. If no
database drivers are available database-related scripts will not work,
but other benchmarks will be functional.

See [README-Oracle.md](README-Oracle.md) for instructions on building
with Oracle client libraries.

# Usage

## General Syntax

The general command line syntax for sysbench is:

		  sysbench [options]... [testname] [command] 

- *testname* is an optional name of a built-in test (e.g. `fileio`,
  `memory`, `cpu`, etc.), or a name of one of the bundled Lua scripts
  (e.g. `oltp_read_only`), or a *path* to a custom Lua script. If no
  test name is specified on the command line (and thus, there is no
  *command* too, as in that case it would be parsed as a *testname*), or
  the test name is a dash ("`-`"), then sysbench expects a Lua script to
  execute on its standard input.

- *command* is an optional argument that will be passed by sysbench to
  the built-in test or script specified with *testname*. *command*
  defines the *action* that must be performed by the test. The list of
  available commands depends on a particular test. Some tests also
  implement their own custom commands.

  Below is a description of typical test commands and their purpose:

	+ `prepare`: performs preparative actions for those tests which need
	them, e.g. creating the necessary files on disk for the `fileio`
	test, or filling the test database for database benchmarks.
	+ `run`: runs the actual test specified with the *testname*
    argument. This command is provided by all tests.
	+ `cleanup`: removes temporary data after the test run in those
    tests which create one.
	+ `help`: displays usage information for the test specified with the
	*testname* argument. This includes the full list of commands
	provided by the test, so it should be used to get the available
	commands.

- *options* is a list of zero or more command line options starting with
	`'--'`. As with commands, the `sysbench testname help` command
	should be used to describe available options provided by a
	particular test.

	See [General command line options](README.md#general-command-line-options)
	for a description of general options provided by sysbench itself.


You can use `sysbench --help` to display the general command line syntax
and options.

## General Command Line Options

The table below lists the supported common options, their descriptions and default values:

*Option*              | *Description* | *Default value*
----------------------|---------------|----------------
| `--threads`           | The total number of worker threads to create                                                                                                                                                                                                                                                                                                                                                                                                                            | 1               |
| `--events`            | Limit for total number of requests. 0 (the default) means no limit                                                                                                                                                                                                                                                                                                                                                                                                      | 0               |
| `--time`              | Limit for total execution time in seconds. 0 means no limit                                                                                                                                                                                                                                                                                                                                                                                                             | 10              |
| `--rate`              | Average transactions rate. The number specifies how many events (transactions) per seconds should be executed by all threads on average. 0 (default) means unlimited rate, i.e. events are executed as fast as possible                                                                                                                                                                                                                                                                 | 0               |
| `--thread-stack-size` | Size of stack for each thread                                                                                                                                                                                                                                                                                                                                                                                                                                           | 32K             |
| `--report-interval`   | Periodically report intermediate statistics with a specified interval in seconds. Note that statistics produced by this option is per-interval rather than cumulative. 0 disables intermediate reports                                                                                                                                                                                                                                                                  | 0               |
| `--debug`             | Print more debug info                                                                                                                                                                                                                                                                                                                                                                                                                                                   | off             |
| `--validate`          | Perform validation of test results where possible                                                                                                                                                                                                                                                                                                                                                                                                                       | off             |
| `--help`              | Print help on general syntax or on a test mode specified with --test, and exit                                                                                                                                                                                                                                                                                                                                                                                          | off             |
| `--verbosity`         | Verbosity level (0 - only critical messages, 5 - debug)                                                                                                                                                                                                                                                                                                                                                                                                                 | 4               |
| `--percentile`        | sysbench measures execution times for all processed requests to display statistical information like minimal, average and maximum execution time. For most benchmarks it is also useful to know a request execution time value matching some percentile (e.g. 95% percentile means we should drop 5% of the most long requests and choose the maximal value from the remaining ones). This option allows to specify a percentile rank of query execution times to count | 95              |

Note that numerical values for all *size* options (like `--thread-stack-size` in this table) may be specified by appending the corresponding multiplicative suffix (K for kilobytes, M for megabytes, G for gigabytes and T for terabytes).

# Versioning

For transparency and insight into its release cycle, and for striving to maintain backward compatibility, sysbench will be maintained under the Semantic Versioning guidelines as much as possible.

Releases will be numbered with the following format:

`<major>.<minor>.<patch>`

And constructed with the following guidelines:

* Breaking backward compatibility bumps the major (and resets the minor and patch)
* New additions without breaking backward compatibility bumps the minor (and resets the patch)
* Bug fixes and misc changes bumps the patch

For more information on SemVer, please visit [http://semver.org/](http://semver.org/).

[coveralls-badge]: https://coveralls.io/repos/github/akopytov/sysbench/badge.svg?branch=1.0
[coveralls-url]: https://coveralls.io/github/akopytov/sysbench?branch=1.0
[travis-badge]: https://travis-ci.org/akopytov/sysbench.svg?branch=1.0
[travis-url]: https://travis-ci.org/akopytov/sysbench?branch=1.0
[license-badge]: https://img.shields.io/badge/license-GPLv2-blue.svg
[license-url]: COPYING
[release-badge]: https://img.shields.io/github/release/akopytov/sysbench.svg
[release-url]: https://github.com/akopytov/sysbench/releases/latest
[deb-badge]: https://img.shields.io/badge/Packages-Debian-red.svg?style=flat
[deb-url]: https://packagecloud.io/akopytov/sysbench?filter=debs
[rpm-badge]: https://img.shields.io/badge/Packages-RPM-blue.svg?style=flat
[rpm-url]: https://packagecloud.io/akopytov/sysbench?filter=rpms


1.0.18使用方式:
# for 1.0.18
oltp_common_option=" --db-driver=mysql --mysql-db=${db} --mysql-user=${user} --mysql-password=${pwd} --mysql-port=${port} --mysql-host=${ip} --tables=${oltp_tables_count} --table_size=${oltp_table_size} --threads=${oltp_tables_count} --create_secondary=${create_secondary} --skip_trx=${oltp_skip_trx} --auto_inc=${oltp_auto_inc} "
oltp_run_option=" --events=${max_requests} --time=${max_time} --report-interval=${report_interval} "

#清理数据
#sysbench ${oltp_common_option} ${oltp_run_option} oltp_read_write cleanup

#导入测试数据, CREATE TABLE `sbtest` (`id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT, `k` INTEGER UNSIGNED, `c` CHAR(120), `pad` CHAR(60));
sysbench ${oltp_common_option} ${oltp_run_option} oltp_read_write prepare

#压测数据
#事务simple-point-select，SELECT c FROM ${rand_table_name} where id=${rand_id};
sysbench ${oltp_common_option} ${oltp_run_option} oltp_point_select run

#事务update-non-index，UPDATE ${rand_table_name} SET c=${rand_str} WHERE id=${rand_id}
sysbench ${oltp_common_option} ${oltp_run_option} oltp_update_non_index run
