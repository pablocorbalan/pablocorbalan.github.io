---
layout: post
title:  "Poster: Building a Stairway to Centralised WSN Control"
date:   2016-02-16
comments: true
---

Smart Cities will have hundreds or thousands (maybe tens of thousands) of wireless sensor nodes deployed that will not only provide valuable data to learn about city resources usage, but will also control city resources. This extends the original Wireless Sensor Network (WSN) data collection application to a more complex scenario in which different network traffic is expected. In such scenario, many users and applications with different requirements may use the network resources deployed. To meet those requirements, we need network-wide management and control to optimise the network behaviour for each application. We need centralised network control.

Centralised routing protocols have been proposed in WSNs by some standards and protocols, such as WirelessHART and Hydro. However, these protocols have some limitations. For instance, WirelessHART is an open standard (but not free) and has limited scalability according to [1]. On the other hand, [Hydro](http://www.cs.berkeley.edu/~stevedh/pubs/smartgrid10hydro.pdf) builds an incomplete network topology model, which limits the network-wide decisions that can be taken.

To address these issues, I am taking a first step towards scalable centralised network control with SMOG. SMOG builds a complete centralised network topology model using probabilistic data structures, specifically, Bloom filters (BFs). If you do not know what a BF is, check out these websites [Bloom Filter by Example - Bill Mill](http://billmill.org/bloomfilter-tutorial/) and [Bloom Filters - Jason Davies](https://www.jasondavies.com/bloomfilter/). SMOG compresses the neighbourhood set of a node in a single BF that is sent to the network controller. The controller upon reception of these BFs can determine with certain False Positive probability the neighbourhood of a node. The usage of BFs makes the network model probabilistic, but tuning the BF size and number of hash functions used, I argue that SMOG can build a very accurate model and still reduce the overhead.

In SMOG, I also studied when new topology reports should be sent by providing three different Modes of Operation - Eventful, Periodic, and Stateful. The MOP choice depends on how reactive we want SMOG to be and how much overhead we can afford. Also, the MOP used plays an important role in the Model Accuracy achieved.

To analyse the behaviour of SMOG, I ran a series of simulations in Cooja to evaluate SMOG's scalability and in the Indriya testbed (100 nodes), to test SMOG's behaviour in a real environment with channel dynamics. So far, SMOG achieves high accuracy with very low overhead in simulations and the testbed under different network conditions. Further evaluation needs to be made to analyse the impact of churn of SMOG.

To read more about SMOG, you can find here the [poster abstract](/docs/posters/ewsn-smog-poster-abstract.pdf) published in [EWSN'16](http://www.iti.tugraz.at/EWSN2016/cms/index.php?id=36) and the [poster](/docs/posters/ewsn-smog-poster.pdf) presented in the conference. You can give me some feedback about your impressions of SMOG, as well as about my 1-Minute Madness presentation in the comments below.

References:

[1] C. Lu, A. Saifullah, B. Li, M. Sha, H. Gonzalez, D. Gunatilaka, C. Wu, L. Nie and Y. Chen, [Real-Time Wireless Sensor-Actuator Networks for Industrial Cyber-Physical Systems](http://www.cse.wustl.edu/%7Elu/papers/pieee-wsan.pdf), Special Issue on Industrial Cyber-Physical Systems, Proceedings of the IEEE, accepted.