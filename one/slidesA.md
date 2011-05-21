!SLIDE title-slide center bullets

# Utilizing Redis in distributed Erlang systems #

* jkvor.com/erlang-factory-london

!SLIDE center smbullets incremental transition=fade

# Jacob Vorreuter and Orion Henry #

![heroku](heroku.png)

* Heroku is a platform composed of heterogeneous components
* Ruby, Erlang, Go
* Components must communicate and share information

!SLIDE center smbullets incremental transition=fade

![redis](redis.png)

* Written in C
* Single threaded
* Non-blocking, evented I/O
* Requires no external dependencies
* Supports data types such as lists, sets and hashes
* Persistence is available via async writes to disk
* Master/slave replication
* Pub/Sub capabilities

!SLIDE smbullets incremental transition=fade

# How Heroku uses Redis #

* As an ephemeral registry of instance health and availability
* As a redundant cache of shared state data
* As a destination for capped collections of log data
* As a pub/sub channel used to generate real-time usage graphs

!SLIDE center smbullets

# github.com/JacobVorreuter/redgrid #

* Automatic Erlang node discovery via Redis

!SLIDE center

![redgrid](redgrid.png)

!SLIDE

	1> node().
	'foo@blah'
	2> redgrid:update_meta([{weight, 0}]).
	ok

	1> node().
	'bar@wha'
	2> redgrid:nodes().
	[{'bar@wha',  [{ip, "10.0.0.2"}]},
	 {'foo@blah', [{ip, "10.0.0.1"},
	               {weight, 0}]}]

!SLIDE center smbullets

# github.com/JacobVorreuter/redo #

* pipelined erlang redis client 

!SLIDE

# Simple, pipelined, no sugar #

        1> redo:start_link().
        {ok,<0.33.0>}

        2> redo:cmd(["SET", "one", "abc"]).
        <<"OK">>

        3> redo:cmd(["SET", "two", "def"]).
        <<"OK">>

        4> redo:cmd([["GET", "one"],
                     ["GET", "two"]]).
        [<<"abc">>,<<"def">>]

!SLIDE transition=fade

        1> bench:sync(1000).
        91ms
        10989 req/sec

        2> bench:sync(10000).
        938ms
        10752 req/sec
 
!SLIDE transition=fade

        1> bench:async(1000, 100).
        38ms
        26315 req/sec

        2> bench:async(10000, 1000).
        294ms
        34482 req/sec

        3> bench:async(34000, 1500).
        1092ms
        31250 req/sec

!SLIDE center transition=fade

# Redis as a redundant cache of shared state #

![hermes](hermes-redis-diag.png)

!SLIDE smbullets transition=fade

* problem: redis slave blocks while loading dataset into memory
* disconnecting from master due to master failure or network hiccup causes slave downtime
* redis log output
* solution: on slave boot, temporarily set correct master auth to allow sync, then unset to prevent re-sync
* slave-sync output

!SLIDE center smbullets

# github.com/JacobVorreuter/nsync

* Erlang Redis replication client

!SLIDE center smbullets

# github.com/JacobVorreuter/tempo #

* Node.js websocket interface to Redis pub/sub channels

