---
title: ATTAK Docs

toc_footers:
  - <a href='#'>Get update notifications</a>

search: true
---

## Contents

* [Introduction](#introduction)
  * [Distributed Topologies](#distributed-topologies)
  * [Why ATTAK](#why-attak)
* [Quick Start](#quick-start)
* [Components](#components)
  * [Topology](#topology)
  * [Processors](#processors)
* [Debugging](#debugging)
  * [Topology Debugger](#topology-debugger)
  * [CLI Simulation](#cli-simulation)

# <a name="introduction"></a>Introduction

Welcome to attak! We aim to create the simplest and most powerful way to build high-scale distributed realtime computation topologies. We provide tools to assist in prototyping, development, deployment, and production monitoring.

## <a name="distributed-topologies"></a>A Distrubted What?

When processing streaming data, it can often be useful to break tasks out into separate microservices. Those microservices usually need to send data to eachother, and good practice dictates some kind of fault-tolerant queue system is used.

A topology is what we call everything together - the set of all data processing microservices (we call them "processors") and data queues ("streams").

## <a name="why-attak"></a>Why use ATTAK

ATTAK has several advantages over existing frameworks like Apache Storm

***Speed and Simplicity***

It's just node (or whatever shell processes you want to call)

Running a local Storm topology can take minutes (Storm has to compile an uber-jar and then boot up the JVM and bolt/spout processes). Simulating an **attak** topology takes seconds. That makes a huge difference in the code -> run -> debug cycle

AWS Lambdas, Kinesis Streams, Google Cloud Functions, PubSub, etc. all have very well established logging and debugging capabilities. Handling producton issues is much easier because ATTAK is built on these proven building blocks

***Price of Prototype***

Apache Storm requires a lot of system resources to run, meaning setting up a prototype/qa/staging deploy can be expensive. [This popular example](https://github.com/nathanmarz/storm-deploy) uses 4 m1-large EC2 instances. Other common deployments (DCOS for example) use even more.

ATTAK's auto-scaling building blocks are pay-per-request, so idle topologies cost nothing.

***Maintenance***

Auto-scaling components mean there are no servers to be juggled, no CPUs or disks to monitor, no logs to rotate, etc. Everything just works...and did we mention it's cheaper?

# <a name="quick-start"></a>Quick Start

### Install cli

`npm install -g attak`

### Create an attak topology

Generate a simple boilerplate project by running:

`attak init`

### Setup environment

Rename `.env.example` to `.env` and put in values as appropriate for your deploy. The required fields are:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_ROLE_ARN`

### Topology simulation and debugging

Visit [the ATTAK ui](http://attak.io#local) and run the simulation with the command displayed, which will look like:

`attak simulate -i [simulation id]`

If you want to run the topology without the UI debugger, simply run

`attak simulate` 

### Deploy the topology

ATTAK will deploy all processors and streams in the topology.

```attak deploy```

# <a name="components"></a>Components

In order to get a topology running you need to build a topology file and define one or more processors.

## <a name="topology"></a>Topology

An topology is simply a JSON structure that defines the features of your application. An ATTAK project is meant to be a node package, so we will `require` the directory, and `index.js` (or whatever is specified in `package.json`) will be loaded.

At its core, a topology is a description of one or more processors and the connections between them. Here's an example of a very simple topology.

```js
module.exports = {
  name: 'attak-example',                          // a topology name is required
  processors: {                                   // declare processors
    reverse: './processors/reverse',
    hello_world_spout: './processors/hello_world'
  },
  streams: [                                      // declare processor connections
    {
      to: 'reverse',                              // simple connection example
      from: 'hello_world_spout'
    }
  ]
}
```

## <a name="processors"></a>Processors

**attak** has a single concept for all data processing units: processors. In the abstract a processor is triggered in response to some event. The processor can access event data if any, and can emit any number of new events. Here's an example processor:

```js
exports.handler = function(event, context, callback) {
  console.log(event);                             // prints 'hello world'
  reversed = event.split('').reverse().join('')   // process event data (reverse it)
  context.emit('reversed strings', reversed)      // emit a "reversed strings" event
  callback()                                      // close up shop
}
```

# <a name="debugging"></a>Debugging

ATTAK provides a number of tools

## <a name="topology-debugger"></a>Topology Debugger

Complex problems require complex solutions, and debugging interaction between distributed components can be extremely difficult without assistance. ATTAK's Topology Debugger is a web tool that help you build and debug your topologies during simulation. It's free to use for everyone.

To use it, visit [the local simulator](http://attak.io#local). You'll see something like this:

[![Debugger ID callout](http://attak.s3.amazonaws.com/local_start.png)](http://attak.io#local)

Running the command from within a project directory will send topology information to the page via local [WebRTC](https://webrtc.org/) connection. The debugger will display your topology as a graph, and then run the simulation.

[![Topology graph example](http://attak.s3.amazonaws.com/graph.png)](http://attak.io#local)

Events emitted during the simulation will be displayed on the debugger in real time. This topology graph shows a processor called `github_events` which has emitted more events than other processors.

The topology debugger is still under heavy development so the specific set of functionality will be documented as it's solidified.

## <a name="cli-simulation"></a>CLI Simulation

`attak simulate`

Simulate pulls in data (from `./input.json` by default) and feeds it through the specified processors. Input data has the following format:

```
{
    "processor_name": {"the_data": "you want to be sent in"},
    "other_processor": "string data is fine",
}
```

Any events emitted from processors will be displayed in blue, and processor console logs will work as normal.

## Deploy a topology

`attak deploy`

Assembles and deploys a series of functions pre-baked with topology information, so event emissions go directly from function to stream to function.

## Trigger a live topology

`attak trigger`

Pulls in data (from `./input.json` by default) and sends it to a live topology instance

# Roadmap

#### Features

- Create basic topologies
    - processors √
    - streams (with topics) √

- Debugging
    - simulate topology locally √
    - trigger live topology √
    - collect topology logs √

- Topology Flow Control
    - managed parallelization (split/join streams)
    - stream aggregation

#### Platforms

_tldr: AWS first_

1. AWS Support
    - [Lambda](https://aws.amazon.com/lambda)/[Kinesis](https://aws.amazon.com/kinesis) topologies √
    - CloudWatch logs √

2. Google Cloud Support
    - [Cloud Functions](https://cloud.google.com/functions/)/[PubSub](https://cloud.google.com/pubsub) topologies
    - StackDriver? logs
  
We intend for full google cloud support, but we won't focus on it until Cloud Functions gets out of alpha. Theoretically that will happen ["soon"](https://github.com/apex/apex/issues/232#issuecomment-218246926)

3. Others? Open an issue! We'd love to hear about your use case.
