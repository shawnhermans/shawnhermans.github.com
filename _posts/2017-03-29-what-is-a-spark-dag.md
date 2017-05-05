---
layout: post
title: "What is DAG in Spark?"
author: Shawn Hermans
date: 2017-03-29 12:00:00 -0600
categories: [spark, dag]
---
Originally posted as an answer to [What is DAG in Spark, and how does it work?](https://www.quora.com/What-is-DAG-in-Spark-and-how-does-it-work/answer/Shawn-Hermans?srid=hLq3)

As Rajagopal ParthaSarathi pointed out, a DAG is a directed acyclic graph. They are commonly used in computer systems for task execution.

In this context, a graph is a collection of nodes that are connected by edges. In the case of Hadoop and Spark, the *nodes* represent executable tasks, and the *edges* are task dependencies. Think of the DAG like a flow chart that tells the system which tasks to execute and in what order. The following is a simple example of an *undirected* graph of tasks.

![undirected graph](https://qph.ec.quoracdn.net/main-qimg-4d31a64225c37d607f102af60602b613)


This graph is undirected because there it does not capture which node is the start node and which is the end node. In other words, this graph does not tell me if the reduce task should be feeding the map tasks or vice versa. The next graph shows a *directed* graph of tasks.

![directed graph of tasks](https://qph.ec.quoracdn.net/main-qimg-9c013afa2c6f72ef2b21d04be194f100.webp)

A directed graph gives an unambiguous direction for each edge. This means that we know that the map tasks feed into the reduce task, rather than the other way around. This property is essential for executing complex workflows since we need to know which tasks should be executed in which order.

Lastly, the graph is *acyclic*, because it does not contain any cycles. A cycle happens when it is possible to loop back to a previous node. Cycles are useful for tasks involving recursion but not as good for large-scale distributed systems. The following are two examples of graphs with cycles.

![cyclic-graph-example1](https://qph.ec.quoracdn.net/main-qimg-059bd48c6284c4be366c0618c81c3588.webp)

![cyclic-graph-example2](https://qph.ec.quoracdn.net/main-qimg-f8c9c17a2204bcdaa17c62c9d21cc030.webp)
