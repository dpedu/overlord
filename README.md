Overlordd
=========

A sane alternative to Circusd and Supervisord

Supervisord quips:

- Can't restart processes forever
- Can't relate processes to one another (z requres x, y)
- No clean way to do one-shot scripts

Circusd quips:

- Can't relate processes to one another (z requres x, y)
- No clean way to do one-shot scripts
- Difficult to install in docker (on ubuntu, requires python-pip apt pkg with gcc, 120MB of deps)

Example config:

(broken into sections)

```
[overlord]
forever=true
configdir=/etc/overlord/
```

This section contains global options for overlord. The above are defaults. Forever means processes should be restarted
forever. All files in the config dir will be processed, alphabetically, as if they were part of the main config.

```
[script:setup]
cmd = /path/to/presetup.sh
```

"Script" are one-shot processes. Scripts will NOT be run unless a daemon asks for it (by depending on it)

```
[daemon:cacheserver]
cmd = /path/to/redis
```

"Daemon"s are process that are meant to be kept running forever.


```
[daemon:myapp]
cmd = ...
deps = setup:always, cacheserver:bounce
```

Another daemon. This one specifies that some other resources should be processed before starting this daemon. Deps is a
comma-separate list of the required resources in the form of `name:state`.

States for daemons

- `ok`: the referenced daemon started successfully
- `bounce`: the referenced daemon started successfully, though if it restarts restart this daemon too

States for scripts:

- `ok`: the referenced script executed successfully (return code == 0)


Options:

- cmd - the command to run. interpreted in a shell.
- dir - working dir to set
- uid - user name to execute as
- gid - group name to execute as

Daemon-only args

- max_age - max process lifetime in seconds. after this duration the process will be killed.
- stop_signal - signal to send when stopping the process (default = TERM)
- respawns - maximum number of times to restart the process
- warmup - how many seconds must the process live to be considered OK
- deps - ordered list of prerequisites to start this daemon. Format is resourcename:resourcestatus
