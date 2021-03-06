# lfecljapp

<a href="resources/images/LispFlavoredErlangClojure-medium-square.png">
<img src="resources/images/LispFlavoredErlangClojure-small-square.png" />
</a>

*An Example LFE/Clojure Multi-node System using OTP and Supervision Trees*


## Introduction

This project is a port of Maxim Molchanov's example Erlang + Clojure interop
project (via JInterface). Only minor changes were made to the Clojure code.
The Erlang code was completely replaced with LFE.

This project demonstrates how one can:

1. Create a Clojure project which communicates with LFE/Erlang nodes
   utilizing [Clojang](https://github.com/clojang/clojang)
   (an Erlang JInterface wrapper)
1. Start a supervised Clojure node in LFE,
1. Send messages to Clojure nodes from LFE
1. Send messages to Clojure nodes from Clojure
1. Send messages to LFE from Clojure
1. Receive and respond to all messages in both Clojure and LFE


### Dependencies

For LFE:

* Erlang
* ``rebar3``

For Clojure:

* Java
* ``lein``

## Building

To get started, compile the LFE source files for `lfenode` and build the
Clojure uberjar for `cljnode`:

```bash
$ make compile
```

## Running

We'll examine two ways of running our application nodes:

* As daemons (non-OTP release)
* From the LFE and Clojure REPLs


### As daemons

Once everything has compiled successfully, you can start an LFE REPL and
then bring the app up:

```
TBD
```

### From the REPLs

During active development it's useful to be able to run your code in a REPL.
Below we show how to do that in the case of `lfecljapp`, starting the two two
servers up from their respective REPLs for easy interaction.


#### 1. Start the Clojure REPL

Start the Clojure REPL and then start the Clojure node's server:

```bash
$ make clojure-repl
```
```
2017-02-12 18:59:34,080 [main] INFO  clojang.agent.startup - Bringing up ...
2017-02-12 18:59:34,267 [main] INFO  clojang.agent.startup - Registered nodes ...
nREPL server started on port 34904 on host 127.0.0.1 - nrepl://127.0.0.1:34904
REPL-y 0.3.7, nREPL 0.2.12
Clojure 1.8.0
OpenJDK 64-Bit Server VM 1.8.0_121-8u121-b13-0ubuntu1.16.04.2-b13
    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
    Exit: Control+D or (exit) or (quit)
 Results: Stored in vars *1, *2, *3, an exception in *e

cljnode.core=>
```


#### 2. Start the Clojure Server

```clj
cljnode.core=> (def server-data (managed-server))
2017-02-13 17:42:23,386 [async-thread-macro-2] INFO  cljnode.server - Starting ...
#'cljnode.core/server-data
```


#### 3. Use the Clojure Server API

Now let's talk to our Clojure server using the Clojure API we created (see
`src/clj/cljnode/api.clj`):

```clj
cljnode.core=> (api/ping server-data)
2017-02-13 17:43:00,081 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
:pong
cljnode.core=> (api/ping server-data)
2017-02-13 17:43:00,947 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
:pong
cljnode.core=> (api/ping server-data)
2017-02-13 17:43:01,632 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
:pong
cljnode.core=> (api/ping server-data)
2017-02-13 17:43:02,377 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
:pong
cljnode.core=> (api/get-ping-count server-data)
2017-02-13 17:43:07,113 [async-thread-macro-1] INFO  cljnode.server - Got :get-ping-count ...
4
```


#### 4. Start LFE

```bash
$ make lfe-repl
```
```
Erlang/OTP 18 [erts-7.3] [source] [64-bit] [smp:4:4] [async-threads:10] ...

   ..-~.~_~---..
  (      \\     )    |   A Lisp-2+ on the Erlang VM
  |`-.._/_\\_.-':    |   Type (help) for usage info.
  |         g |_ \   |
  |        n    | |  |   Docs: http://docs.lfe.io/
  |       a    / /   |   Source: http://github.com/rvirding/lfe
   \     l    |_/    |
    \   r     /      |   LFE v1.3-dev (abort with ^G)
     `-E___.-'

(lfenode@liberator)lfe>
```


#### 5. Use the LFE Client API (for the Clojure Server)

```cl
(lfenode@liberator)lfe> (api:ping 'cljnode@liberator)
pong
(lfenode@liberator)lfe> (api:ping 'cljnode@liberator)
pong
(lfenode@liberator)lfe> (api:ping 'cljnode@liberator)
pong
(lfenode@liberator)lfe> (api:ping 'cljnode@liberator)
pong
(lfenode@liberator)lfe> (api:get-ping-count 'cljnode@liberator)
8
```

If you take a peek back over in the Clojure terminal, you should see log
messages for each of those API calls:

```clj
2017-02-14 00:44:06,717 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
2017-02-14 00:44:07,623 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
2017-02-14 00:44:08,599 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
2017-02-14 00:44:13,694 [async-thread-macro-1] INFO  cljnode.server - Got :ping ...
2017-02-14 00:44:15,338 [async-thread-macro-1] INFO  cljnode.server - Got :get-ping-count ...
```


#### 6. Shutdown the Server

You can do this either in Clojure:

```clj
cljnode.core=> (api/stop server-data)
:stopping
```

Or LFE:

```cl
(lfenode@liberator)lfe> (api:stop 'cljnode@liberator)
stopping
```

The end result will be the same:

```clj
2017-02-14 01:08:08,664 [async-thread-macro-1] WARN  cljnode.server - Got :stop ...
2017-02-14 01:08:08,875 [async-dispatch-3] INFO  cljnode.core - Server stopped ...
```


## Example Code

### Clojure Server

```clj
(defn run
  [cmd-chan]
  (log/info "Starting Clojure node with nodename ="
            (System/getProperty "node.sname"))
  (let [init-state 0]
    (loop [png-count init-state]
      (match (receive)
        [:register caller]
          (do
            (log/infof "Got :register request from %s ..." caller)
            (mbox/link (self) caller)
            (! caller :linked)
            (recur png-count))
        [:ping caller]
          (do
            (log/infof "Got :ping request from %s ..." caller)
            (! caller :pong)
            (recur (inc png-count)))
        [:get-ping-count caller]
          (do
            (log/infof "Got :get-ping-count request from %s ..."  caller)
            (! caller png-count)
            (recur png-count))
        [:stop caller]
          (do
            (log/warnf "Got :stop request from %s ..." caller)
            (! caller :stopping)
            :stopped)
        [:shutdown caller]
          (do
            (log/warnf "Got :shutdown request from %s ..." caller)
            (! caller :shutting-down)
            (async/>! cmd-chan :shutdown))
        [_ caller]
          (do
            (log/error "Bad message received: unknown command")
            (! caller [:error :unknown-command])
            (recur png-count))
        [_]
          (do
            (log/error "Bad message received: improperly formatted")
            (recur png-count))))))
```


### Clojure API

```clj
(defn send-only
  [server-data msg]
  (async/>!! (get-in server-data [:bridge :channel]) msg)
  :ok)

(defn send-and-receive
  [server-data msg]
  (send-only server-data msg)
  (receive (get-in server-data [:bridge :mbox])))

(defn register
  [server-data]
  (send-only server-data :register))

(defn ping
  [server-data]
  (send-and-receive server-data :ping))

(defn get-ping-count
  [server-data]
  (send-and-receive server-data :get-ping-count))

(defn stop
  [server-data]
  (send-and-receive server-data :stop))
```


### LFE API

```cl
(defun send-only (node-name msg)
  (! `#(default ,node-name) `#(,msg ,(self))))

(defun send-and-receive (node-name msg)
  (send-only node-name msg)
  (receive
    (data data)))

(defun register (node-name)
  (send-only node-name 'register))

(defun ping (node-name)
  (send-and-receive node-name 'ping))

(defun get-ping-count (node-name)
  (send-and-receive node-name 'get-ping-count))

(defun stop (node-name)
  (send-and-receive node-name 'stop))
```


## Fun for the Future

Here are some things I'd like to play with in this project:

* Setting up some example long-running computations in Clojure.
* Spawn multiple nodes and distribute computations across an LFE cluster.
* Use ``lfecljapp`` as a proxy to a Storm cluster, and explores ways in which
  it might be useful to interact with Storm from LFE.
