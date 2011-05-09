!SLIDE title-slide

# Utilizing Redis in distributed Erlang systems #

!SLIDE

# Jacob Vorreuter and Orion Henry #

![heroku](heroku.png)

Heroku is a platform composed of heterogeneous components (Ruby, Erlang, Go)

These components must communicate and share information

!SLIDE

# Enter Redis... #

* Extremely fast
  * written in C
  * single threaded
  * non-blocking, evented I/O
  * requires no external dependencies (not even an event library like libevent/libev)
* Supports data types such as lists, sets and hashes
* Persistence is available via async writes to disk
* Master/slave replication
* Pub/Sub capabilities

!SLIDE

# How Heroku uses Redis #

* As a redundant cache of shared state data
* To track running instances
* To collect statistics across applications
* To collect logs from multiple applications

!SLIDE

* First case: As a redundant cache of shared state data

Hermes
* Erlang HTTP proxy
* Balances HTTP requests across your app''s backend web server processes, tracking load and intelligently routing traffic to available resources

!SLIDE center

![hermes](hermes-redis-diag.png)

!SLIDE

* problem: redis slave blocks while loading dataset into memory
disconnecting from master due to master failure or network hiccup causes slave downtime

redis log output

* solution: on slave boot, temporarily set correct master auth to allow sync, then unset to prevent re-sync

slave-sync output


