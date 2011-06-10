!SLIDE title-slide center bullets

# Utilizing Redis in distributed Erlang systems #

* erlang-factory.herokuapp.com

!SLIDE center smbullets incremental transition=scrollUp

# Jacob Vorreuter and Orion Henry #

![heroku](heroku.png)

* Heroku is a platform composed of heterogeneous components
* Ruby, Erlang, Go
* Components must communicate and share information

!SLIDE center smbullets incremental transition=scrollUp

![redis](redis.png)

* Written in C
* Single threaded
* Non-blocking, evented I/O
* Requires no external dependencies
* Supports data types such as lists, sets and hashes
* Persistence is available via async writes to disk
* Master/slave replication
* Pub/Sub capabilities

!SLIDE smbullets incremental transition=scrollUp

# How Heroku uses Redis #

* As an ephemeral store of instance health and availability
* As a redundant cache of shared state data
* As a destination for capped collections of log data
* As a pub/sub channel powering real-time usage graphs

!SLIDE center transition=scrollUp

# Tools and examples #

!SLIDE center smbullets transition=scrollUp

# github.com/JacobVorreuter/redo #

* pipelined erlang redis client 

!SLIDE smbullets transition=scrollUp

# First there was redis_pool #

* only accepted raw multi bulk redis commands
* defined pools of gen_servers holding connections to Redis
* gen_server:call(redis_pool:pid(), {cmd, Cmd}, ?TIMEOUT)
* each connection was syncronous
* Redis could only process a # of commands equal to # of pids in pool

!SLIDE transition=scrollUp small

# Redo: simple, pipelined, no sugar #

        1> redo:start_link().
        {ok,<0.33.0>}

        2> {ok, Pid} = redo:start_link(undefined).
        {ok,<0.37.0>}        

        3> redo:cmd(["SET", "one", "abc"]).
        <<"OK">>

        4> redo:cmd(Pid, ["SET", "two", "def"]).
        <<"OK">>

        5> redo:cmd([["GET", "one"],
                     ["GET", "two"]]).
        [<<"abc">>,<<"def">>]

!SLIDE transition=scrollUp small

# More commands #

        5> redo:cmd([["SADD", "sfoo", "123"],
                     ["SADD", "sfoo", "456"]]).
        [1,1]

        6> redo:cmd(["SMEMBERS", "sfoo"]).
        [<<"123">>,<<"456">>]

        7> redo:cmd(["HMSET", "hfoo",
                     "width", "100",
                     "height", "75",
                     "depth", "50"]).
        <<"OK">>

        8> redo:cmd(["HGETALL", "hfoo"]).
        [<<"width">>,<<"100">>,
         <<"height">>,<<"75">>,
         <<"depth">>,<<"50">>]
        

!SLIDE transition=scrollUp

# Fast #

        1> bench:sync(1000).
        91ms
        10989 req/sec

        2> bench:sync(10000).
        938ms
        10752 req/sec
 
!SLIDE transition=scrollUp

# Faster #

        1> bench:async(1000, 100).
        38ms
        26315 req/sec

        2> bench:async(10000, 1000).
        294ms
        34482 req/sec

!SLIDE center smbullets transition=scrollUp

# github.com/JacobVorreuter/redgrid #

* Automatic Erlang node discovery via Redis

!SLIDE center transition=scrollUp

![redgrid2](redgrid2.png)

!SLIDE center transition=scrollUp

![redgrid](redgrid.png)

!SLIDE transition=scrollUp small

# Attaching meta data to nodes #

	1> node().
	'foo@blah'
	2> redgrid:update_meta([{weight, 0}]).
	ok

	1> node().
	'bar@blah'
	2> redgrid:nodes().
	[{'bar@blah',  [{ip, "10.0.0.2"}]},
	 {'foo@blah', [{ip, "10.0.0.1"},
	               {weight, 0}]}]

!SLIDE center smbullets transition=scrollUp

# github.com/JacobVorreuter/nsync

* Erlang Redis replication client

!SLIDE center smbullets transition=scrollUp

# A common problem at Heroku #

* State must be shared between components or nodes. Maintaining a local cache of state data is slow to initially populate and difficult to keep in sync

!SLIDE center smbullets incremental transition=scrollUp

# Redis replication #

* slave opens a tcp socket connected to the master redis
* slave issues a "SYNC" command
* master asynchronously dumps its dataset to disk
* dataset is transfered to the slave as an rdb dump
* slave loads dataset into memory
* large strings must be decompressed
* master streams updates to the slave using the redis text protocol

!SLIDE smbullets transition=scrollUp

# Nysnc #

* under 700 loc (majority is parsing rdb dump format)
* uses lzf compression library via NIF
* implements a callback interface

!SLIDE transition=scrollUp small

# Nsync callback structure #

        -module(myapp).
        -export([nsync_callbacks/1]).

        nsync_callbacks({load, K, V}) ->
            ...

        nsync_callbacks({cmd, Cmd, Args}) ->
            ...

        1> nsync:start_link([
            {host, "localhost"},
            {port, 6379},
            {callback, {myapp, nsync_callbacks, []}}
        ]).

!SLIDE transition=scrollUp

# Nsync callback structure #

        {load, Key, Value}
        {load, eof}
        {cmd, Cmd, Args}
        {error, closed}

!SLIDE smbullets incremental transition=scrollUp

# github.com/heroku/logplex #

* Heroku log router
* Receives syslog packets
* from Heroku infrastructure components
* from user applications

!SLIDE center transition=scrollUp

![logs](logs.png)

!SLIDE center transition=scrollUp

# Logplex replication #

![logplex](logplex_nsync.png)

!SLIDE smbullets transition=scrollUp
      
# Hermes #

* Heroku HTTP router
* resolve Host header -> ip/port of suitable dyno
* must maintain accurate picture of process states

!SLIDE center transition=scrollUp

# Redis as a redundant cache of shared state #

![hermes](hermes-redis-diag.png)

!SLIDE smbullets transition=scrollUp

# One limitation of Redis replication #

* Dataset is unavailable during slave resync
* Network partition or hiccup causes downtime
* (error) LOADING Redis is loading the dataset in memory

!SLIDE transition=scrollUp smaller

# redis.log #

        - Reading from client: Connection reset by peer
        * Connecting to MASTER...
        * MASTER <-> SLAVE sync started: SYNC sent
        * MASTER <-> SLAVE sync: receiving 66055964 bytes from master
        * MASTER <-> SLAVE sync: Loading DB in memory
        * MASTER <-> SLAVE sync: Finished with success

!SLIDE transition=scrollUp small

# slave-sync.sh #

        00:24:57: CONFIG SET MASTERAUTH password
        00:24:57: OK
        00:24:57: STATUS: master_link_status:down
        00:24:57: waiting for master_link_status:up
        ...
        00:25:08: STATUS: master_link_status:up
        00:25:08: CONFIG SET MASTERAUTH NULL
        00:25:08: OK

!SLIDE center smbullets transition=scrollUp

# github.com/JacobVorreuter/tempo #

* Node.js websocket interface to Redis pub/sub channels

!SLIDE center transition=scrollUp

![tempo](tempo.png)

!SLIDE center transiton=scrollUp

# erlang-factory.herokuapp.com #
