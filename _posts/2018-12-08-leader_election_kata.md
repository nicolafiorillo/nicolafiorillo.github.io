---
layout: post
title: Kata in Elixir - The Leader Election
tags:
- Elixir
- kata
- blockchain
---

## Kata description
Think of a set of K (K > 1) connected nodes that form together a distributed system.
This task is about implementing an algorithm that allows selecting a Leader from those nodes. There can only be one Leader at each point in time, and if Leader is unavailable, stopped, died or disappeared in a singularity the algorithm should elect a new Leader as soon as possible.

## Requirements
There are two major parts in the implementation:

### Current Leader Monitoring
Once in T seconds, each node sends a message PING to the Leader. If no response is received in 4xT seconds then the current Leader is considered dead and the node that sent that message begins a new Leader election.

### Leader Election
All nodes know each other (each node knows address and port of any other node). Each node has a unique identifier and identifiers are sortable. There are 3 types of messages: `ALIVE?` , `FINETHANKS`, `IAMTHEKING`.

1. The node that starts the election sends a message `ALIVE?` to all nodes that have ID greater than ID of the current node.
    * If nobody responded `FINETHANKS` in T seconds then the current node becomes the Leader and sends out messages `IAMTHEKING`.
    * If the current node received `FINETHANKS` then it waits for T seconds the message `IAMTHEKING` and if it's not received then election process is started again.

2. If a node receives `ALIVE?` then it responds with `FINETHANKS` message and immediately starts the election.
    * If the node that received `ALIVE?` has the biggest ID then it immediately sends out `IAMTHEKING` message.
3. If a node receives `IAMTHEKING` then it remembers the sender of this message as the Leader.
4. If a node joins the system then it immediately initiates the election.

## Solution and code
[https://github.com/nicolafiorillo/leader_election](https://github.com/nicolafiorillo/leader_election)
