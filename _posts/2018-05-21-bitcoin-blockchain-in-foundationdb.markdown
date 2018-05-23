---
layout: post
title: "Bitcoin blockchain in FoundationDB"
date: 2018-05-22
tags: foundationdb, blockchain, bitcoin
permalink: /post/bitcoin-blockchain-in-foundationdb/
uid: 92f1588f-9232-427f-bc58-6ffca1cd1b69
---
As I decided some time ago that my site - [chaindex.com](https://chaindex.com/) - will offer simple [blockchain explorer](https://chaindex.com/blockchain/) after considering some available solutions, I decided to build my own blockchain database for querying BTC addresses' balances.

Clean [Bitcoin Core](https://bitcoin.org/en/download) does not allow for querying arbitrary addresses unless they have been added to local wallet. This makes `bitcoind` unsuitable for checking random addresses.

I've tried using [Bitpay insight](https://github.com/bitpay/insight) which uses modified `bitcoind` internally with additional indexes on addresses. I dropped it because it didn't (at least at the time) support neither [SegWit](https://en.bitcoin.it/wiki/Segregated_Witness) addresses nor new [Bech32](https://en.bitcoin.it/wiki/Bech32) address format and also choked on more popular addresses.

I built simplified Bitcoin Node which leached and parsed blockchain data from dedicated Bitcoin Core full node and tools to feed this data into structurized database that allow for fast address queries.

My first choice was [PostgreSQL](https://www.postgresql.org). After many optimization I belive I squeezed anything possible from this marvelous RDBMS. I used indexes (obviously) and made heavy use of partitioning and table inheritance to keep indexing robust. I've tried bulk inserts and creating indexes after loading all the data. I've optimized server settings and used SSDs to make things faster. I have eventually succeeded but it took many weeks of trials and failures. The database worked but was so big and slow that inserting another fresh blocks as they were mined, took little below 10 minutes.

After exhausting and semi-succesful attempt with PostgreSQL I started leaning towards noSQL solutions. My first choice was [LevelDB](http://leveldb.org) obviously because Bitcoin Core uses it internally to store blockchain data in full nodes but I quickly switched to [RocksDB](https://rocksdb.org) as it appeared at least 4 times faster in loading data compared to LevelDB.

With RocksDB and some additional address cache in PostgreSQL database, I've created solution that is being used currently at [chaindex.com](https://chaindex.com). It performs very quickly and freshly mined blocks are added in a matter of seconds. Data is held on low-end cheap VPS with 24GB RAM and SSD storage. Today (we're at block 523684) - whole database uses 275GB of disk space. I takes around 8 seconds to process new block and it requires 5 days to load whole bitcoin blockchain from scratch.

## Enter FoundationDB

Some time ago I stumbled upon news about releasing [FoundationDB](https://www.foundationdb.org) as Open Source by Apple. I had never heard about it before but apparent project maturity and use cases in the wild made it very compelling. My enthusiasm has been tempered by the lack of native node.js bindings but situation improved soon thanks to [foundatiobdb](https://www.npmjs.com/package/foundationdb) npm package.

## Client in docker container

In its current form, package requires native client to be installed. As I do all my development on macOS I could have used native macOS package available [here](https://www.foundationdb.org/download/), but I went with custom [Docker CE](https://www.docker.com/community-edition), Ubuntu based container sporting node.js and FoundationDB client.

Example `Dockerfile` could look like this (do not use it verbatim):

```
FROM ubuntu:16.04
MAINTAINER Jakub Fedyczak <jakub@fedyczak.net>

ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

ADD https://www.foundationdb.org/downloads/5.1.7/ubuntu/installers/foundationdb-clients_5.1.7-1_amd64.deb /client.deb
RUN apt-get update && apt-get install -y python curl
RUN dpkg -i /client.deb && rm /*.deb

ADD https://deb.nodesource.com/setup_9.x /node.sh
RUN bash /node.sh && rm /node.sh
RUN apt-get install -y nodejs

CMD ["/bin/bash", "/home/app/src/start.sh"]

# UID is chosen arbitrary to match host system user - fixes issues with docker volumes permissions
RUN useradd -u 12582 -m -s /bin/bash app

RUN mkdir -p /home/app/src/node_modules /home/app/src/data
RUN chown -R app:app /home/app/src && chmod -R 700 /home/app/src/node_modules /home/app/src/data
WORKDIR /home/app/src
USER app

RUN echo "export TERM=xterm-color" >> /home/app/.bashrc

VOLUME /home/app/src/node_modules
VOLUME /home/app/src/data
```

## FoundationDB server configuration

I'm using two SSD VPS machines running Ubuntu 16.04 with 3 fdb procesess on each VPS, forming one cluster.

Filesystems are mounted with `defaults, noatime, discard` options to optimize SSD utilization.

I've added following lines to `/etc/foundationdb/` - one line per running process.

```
[fdbserver.4500]
[fdbserver.4501]
[fdbserver.4502]
```

After starting FDB on first server, you have to copy `/etc/foundationdb/fdb.cluster` file from first machine to second one to automatically add second server to the cluster.

After starting up the cluster, I've converted it to `ssd` storage using `configure ssd` command in `fdbcli`. The default `memory` storage engine requires all the data to fit into RAM and uses disk drive only as a permanent storage. The `ssd` mode makes FDB use SSD drive as a primary store and allows for larger then RAM databases.

You can check current cluster status using `fdbcli` interface:

```
# fdbcli 
Using cluster file `fdb.cluster'.

The database is available.

Welcome to the fdbcli. For help, type `help'.
fdb> status details

Using cluster file `fdb.cluster'.

Configuration:
  Redundancy mode        - single
  Storage engine         - ssd-2
  Coordinators           - 1

Cluster:
  FoundationDB processes - 6
  Machines               - 2
  Memory availability    - 4.0 GB per process on machine with least available
                           >>>>> (WARNING: 4.0 GB recommended) <<<<<
  Retransmissions rate   - 9 Hz
  Fault Tolerance        - 0 machines
  Server time            - 05/22/18 15:47:42

Data:
  Replication health     - Healthy (Repartitioning.)
  Moving data            - 0.020 GB
  Sum of key-value sizes - 48.080 GB
  Disk space used        - 60.044 GB

Operating space:
  Storage server         - 250.9 GB free on most full server
  Log server             - 252.0 GB free on most full server

Workload:
  Read rate              - 785 Hz
  Write rate             - 885 Hz
  Transactions started   - 7 Hz
  Transactions committed - 1 Hz
  Conflict rate          - 0 Hz

Backup and DR:
  Running backups        - 0
  Running DRs            - 0

Process performance details:
  xxx.xxx.xxx.x:4500     ( 10% cpu; 17% machine; 0.004 Gbps; 57% disk IO; 2.9 GB / 4.0 GB RAM  )
  xxx.xxx.xxx.x:4501     ( 12% cpu; 17% machine; 0.004 Gbps; 80% disk IO; 2.8 GB / 4.0 GB RAM  )
  xxx.xxx.xxx.x:4502     ( 11% cpu; 17% machine; 0.004 Gbps; 60% disk IO; 2.8 GB / 4.0 GB RAM  )
  yyy.yyy.yyy.yy:4500    ( 19% cpu; 26% machine; 0.005 Gbps; 99% disk IO; 2.9 GB / 4.2 GB RAM  )
  yyy.yyy.yyy.yy:4501    ( 28% cpu; 26% machine; 0.005 Gbps; 99% disk IO; 2.9 GB / 4.2 GB RAM  )
  yyy.yyy.yyy.yy:4502    ( 21% cpu; 26% machine; 0.005 Gbps; 99% disk IO; 2.9 GB / 4.2 GB RAM  )

Coordination servers:
  xxx.xxx.xxx.x:4500  (reachable)

Client time: 05/22/18 15:47:41

fdb> _
```

I will update this text after loading full blockchain data into the cluster.

