- Feature Name: irodsMonitor
- Status: draft
- Start Date: YYYY-MM-DD
- Authors: Terrell Russell
- RFC PR: (PR # after acceptance of initial draft)
- iRODS Issue:

# Summary


This RFC proposes a new binary to be run alongside each irodsServer process.  This process would provide information about the server to interested parties (aggregators, dashboards, etc.).  The irodsMonitor process would provide JSON via read-only REST endpoints.  Having a built-in monitoring tool would allow easier integration of iRODS into existing robust network topologies with monitoring solutions already in place.  The information provided by the irodsMonitor processes would reduce/remove the need to deploy 'Nagios'-style pings and interpretation for monitoring iRODS by providing them out-of-the-box.


# Motivation

The iRODS community has recently shared multiple examples of how they are monitoring iRODS within their infrastructure.  There is a clamoring for consistency and a voice from the server itself on how it is doing rather than looking at the shadow of its behavior on the network.

An irodsMonitor process would support heartbeat monitoring, freespace information, resource/storage health, network connection information and other basic information that would allow a dashboard to relate to the sysadmins about the health of the system.  Additionally, irodsServers that hold no storage (e.g. catalog providers behind a load balancer) would now appear in reports about the Zone (they are currently hidden from izonereport output). The expected outcome for a particular REST query would be a JSON response with all pertinent information encoded in a standard way - formalized by a schema which would be published by the iRODS Consortium.


## Detailed design

A new irodsMonitor binary to be built/packaged/tested/supported.  It will not (?) affect any existing server processes or implementation.

It will run alongside (or as a child of) the irodsServer process.

irodsctl should also be aware and capable of starting/stopping irodsMonitor.

This binary would provide read-only JSON responses to RESTful queries.

There will be some specific configuration in server_config.json and policy set for the irodsMonitor, including but not limited to:
 - gossip port (default 1249)
 - gossip interval (default 10s?)
 - active resource monitoring consecutive check threshold (default 2)

At first, a couple endpoints:

/heartbeat => is the server awake

/status => general status/stats

Potentially including all of the following:

- uptime
- number of current irodsAgents
- irodsReServer status
- current network utilization (sending/receiving)
- storage resource health (up/down/freespace)
- ping times to other iRODS servers
- bandwidth stats to other iRODS servers
- existing metalnx service stats
- delayed queue statistics

Not all of these items would need to be included in a first release.

This irodsMonitor could be written in any language - but probably it should be C++ or Python.


## Drawbacks

Running an additional server binary (and service) provides a new vector for network attackers.  As this is intended to be read-only, the risk is minimal and far outweighed by the utility of giving insight into the health of the iRODS network.

We run the risk of providing misleading or incomplete information to a sysadmin giving someone false alarms or false confidence.

An additional server providing this information will use minimal additional resources on the server.  There should probably be a way to configure the system *not* to run the irodsMonitor (but we will need to think through the ramifications of how that affects all downstream aggregators/dashboards).


## Rationale and Alternatives

This section is extremely important. See the
[README](README.md#rfc-process) file for details.

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

This irodsMonitor will bring a missing piece of being a good network citizen under the umbrella of the iRODS server software.  There have been other solutions cobbled together with new binaries (iping) as well as custom Nagios scripts to expect certain behavior from the irodsServers.  These have been less than compelling as they were third-party tools not under testing by the Consortium, they were brittle under load, and they still could not gather the information they wanted (resource health, %-full, etc.).

We have considered writing and supporting Nagios plugins, but those would still only be able to get a subset of the interesting information sysadmins want from within the iRODS ecosystem.

The impact of not providing this irodsMonitor is that sysadmins will continue to question the placement of iRODS within their enterprise-class infrastructure.  Providing this kind of information is expected and should be included by iRODS upon installation/configuration.


## Unresolved questions

  
- Is this a standalone separate binary that is stood up at the same time as irodsServer?  Or is it a child of irodsServer?
- What default port does this listen on?  Where is that configured?
- What language is this written in?
- What are the endpoints?
- What are the responses?
- Where are these pieces of information stored?  In the catalog?  What are the dependencies?
