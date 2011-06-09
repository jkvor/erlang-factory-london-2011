## Utilizing Redis is distributed Erlang systems

Jacob Vorreuter and Orion Henry

## Code snippets

### Redo

    redo:start_link().
    {ok, Pid} = redo:start_link(undefined).
    redo:cmd(["SET", "one", "abc"]).
    redo:cmd(Pid, ["SET", "two", "def"]).
    redo:cmd([["GET", "one"], ["GET", "two"]]).
    redo:cmd([["SADD", "sfoo", "123"], ["SADD", "sfoo", "456"]]).
    redo:cmd(["SMEMBERS", "sfoo"]).
    redo:cmd(["HMSET", "hfoo", "width", "100", "height", "75", "depth", "50"]).
    redo:cmd(["HGETALL", "hfoo"]).

### Redgrid

    $ erl -pa ebin deps/redo/ebin -sname foo
    (foo@Jacob-Vorreuters-MacBook-Pro)1> redgrid:start_link().
    {ok,<0.39.0>}

    $ erl -pa ebin deps/redo/ebin -sname bar
    (bar@Jacob-Vorreuters-MacBook-Pro)1> redgrid:start_link().
    {ok,<0.39.0>}

    (foo@Jacob-Vorreuters-MacBook-Pro)2> [node()|nodes()].
    ['foo@Jacob-Vorreuters-MacBook-Pro',
     'bar@Jacob-Vorreuters-MacBook-Pro']
    (foo@Jacob-Vorreuters-MacBook-Pro)3> redgrid:nodes().
    [{'foo@Jacob-Vorreuters-MacBook-Pro', [<<"ip">>, <<"localhost">>]},
     {'bar@Jacob-Vorreuters-MacBook-Pro', [<<"ip">>, <<"localhost">>]}]

    (bar@Jacob-Vorreuters-MacBook-Pro)2> redgrid:update_meta([{weight, 50}]).

    (foo@Jacob-Vorreuters-MacBook-Pro)4> redgrid:nodes().
    [{'foo@Jacob-Vorreuters-MacBook-Pro',[<<"ip">>, <<"localhost">>]},
     {'bar@Jacob-Vorreuters-MacBook-Pro',[{<<"weight">>,<<"50">>},
                                          {<<"ip">>,<<"localhost">>}]}]


### Logplex\_face

http://github.com/JacobVorreuter/logplex_face/blob/redgrid/logplex_face.rb

### Nsync

    1> nsync:start_link().
    {ok,<0.33.0>}
    nsync {load,<<"sfoo">>,["123","456"]}
    nsync {load,<<"hfoo">>,{dict,3,16,16,8,80,48,{[],[],[],[],[],[],[],[],[],[],[],[],[],[],[],[]},{{[],[],[],[],[],[[<<"height">>|<<"75">>]],[],[],[[<<"depth">>|<<"50">>]],[],[],[],[],[],[],[[<<"width">>|<<"100">>]]}}}}
    nsync {load,<<"two">>,<<"def">>}
    nsync {load,<<"one">>,<<"abc">>}
    nsync {load,eof}

    1> redo:start_link().

    2> [begin timer:sleep(1000), redo:cmd(["SET", "next_key", N]) end || N <- lists:seq(1, 15)].

    3> redo:cmd(["SMOVE", "sfoo", "sbar", "123"]).
    
    
    
    
