- Feature Name: irodsMonitor
- Status: draft
- Start Date: YYYY-MM-DD
- Authors: Terrell Russell
- RFC PR: (PR # after acceptance of initial draft)
- iRODS Issue:

# Summary


This RFC proposes a new binary to be run alongside each irodsServer process.  This process would provide information about the server to interested parties (aggregators, dashboards, etc.).  The irodsServer process would provide JSON via read-only REST endpoints.  Having a built-in monitoring tool would allow easier integration of iRODS into existing robust network topologies with monitoring solutions already in place.  The information provided by the irodsMonitor processes would reduce/remove the need to deploy 'Nagios'-style pings and interpretation for monitoring iRODS by providing them out-of-the-box.


# Motivation

The iRODS community has recently shared multiple examples of how they are monitoring iRODS within their infrastructure.  There is a clamoring for consistency and a voice from the server itself on how it is doing rather than looking at the shadow of its behavior on the network.

An irodsMonitor process would support heartbeat monitoring, freespace information, resource/storage health, network connection information and other basic information that would allow a dashboard to relate to the sysadmins about the health of the system.  The expected outcome for a particular REST query would be a JSON response with all pertinent information encoded in a standard way - formalized by a schema which would be published by the iRODS Consortium.


# Guide-level explanation

How do we teach this?

Explain the proposal as if it was already included in the project and
you were teaching it to another iRODS developer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples. Take into account that a product manager (PM) will want to connect back the work introduced by the RFC with user stories. Whenever practical, do ask PMs if they already have user stories that relate to the proposed work, and do approach PMs to attract user buy-in and mindshare if applicable.
- Explaining how iRODS contributors and users should think about
  the feature, and how it should impact the way they use
  iRODS. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to
  existing users and new users.

For implementation-oriented RFCs (e.g. for core internals), this
section should focus on how contributors should think about
the change, and give examples of its concrete impact. For policy RFCs,
this section should provide an example-driven introduction to the
policy, and explain its impact in concrete terms.

---

TODO


# Reference-level explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

(You may replace the section title if the intent stays clear.)

- Its interaction with other features is clear.
- It covers where this feature may be surfaced in other areas of the product
   - If the change influences a user-facing interface, make sure to preserve consistent user experience (UX). Prefer to avoid UX changes altogether unless the RFC also argues for a clear UX benefit to users. If UX has to change, then prefer changes that match the UX for related features, to give a clear impression to users of homogeneous CLI / GUI elements. Avoid UX surprises at all costs. If in doubt, ask for input from other engineers with past UX design experience and from your design department.
- It considers how to monitor the success and quality of the feature.
   - Your RFC must consider and propose a set of metrics to be collected, if applicable, and suggest which metrics would be useful to users and which need to be exposed in a public interface.
   - Your RFC should outline how you propose to investigate when users run into related issues in production. If you propose new data structures, suggest how they should be checked for consistency. If you propose new asynchronous subsystems, suggest how a user can observe their state via tracing. In general, think about how your coworkers and users will gain access to the internals of the change after it has happened to either gain understanding during execution or troubleshoot problems.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous
section, and explain more fully how the detailed proposal makes those
examples work.

---

TODO


## Detailed design

A new irodsMonitor binary to be built/packaged/tested/supported.  It will not (?) affect any existing server processes or implementation.

It will run alongside (or as a child of) the irodsServer process.

irodsctl should also be aware and capable of starting/stopping irodsMonitor.

This binary would provide read-only JSON responses to RESTful queries.


At first, a couple endpoints:

/heartbeat => is the server awake

/status => general status/stats

Potentially including all of the following:

- uptime
- number of current agents
- current network utilization (sending/receiving)
- storage resource health (up/down/freespace)
- ping times to other iRODS servers
- bandwidth stats to other iRODS servers
- existing metalnx service stats

Not all of these items would need to be included in a first release.

This irodsMonitor could be written in any language - but probably it should be C++ or Python.


## Drawbacks

Running an additional server binary (and service) provides a new vector for network attackers.  As this is intended to be read-only, the risk is minimal and far outweighed by the utility of giving insight into the health of the iRODS network.

We run the risk of providing misleading or incomplete information to a sysadmin giving someone false alarms or false confidence.

An additional server providing this information will use minimal additional resources on the server.  There should probably be a way to configure the system *not* run the rodsMonitor (but we will need to think through the ramifications of how that affects all downstream aggregators/dashboards).


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
