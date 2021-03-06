+++
pre = "<b>3.3.3. </b>"
toc = true
title = "Supported Registry Centers"
weight = 3
+++

## SPI
[Service Provider Interface (SPI)](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html) is an API intended to be implemented or extended by a third party. It can be used to enable framework extension and replaceable components.

ShardingSphere loads the registry by using SPI for database orchestration to do circuit breaking and disable databases. Currently, ShardingSphere supports two popular registries, Zookeeper and Etcd. In addition, you can use other third-party registries and inject them into ShardingSphere via SPI for database orchestration.

When you need to use the ShardingSphere built-in registry implementation solutions, Maven artifactId should be added to introduce the corresponding registry implementation solution for databases orchestration.

## Zookeeper

ShardingSphere adopts [Apache Curator](http://curator.apache.org/) as Zookeeper implementation solution. Please use Zookeeper 3.4.6 and above, and refer to [official website](https://zookeeper.apache.org/).

## Etcd

ShardingSphere adopts [Etcd](https://coreos.com/etcd/) as Etcd implementation solution. Please use Etcd V3 and above, and refer to [official website](https://coreos.com/etcd/docs/latest).

## Others

Please implement the logic coding using SPI.
