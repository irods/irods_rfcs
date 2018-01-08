- Feature Name: Logging Refactor
- Status: draft
- Start Date: YYYY-MM-DD
- Authors: Terrell Russell
- RFC PR: (PR # after acceptance of initial draft)
- iRODS Issue: 


# Summary

iRODS logging as currently implemented is inconsistent, incomplete, and not helpful at following an error or connection behavior across multiple servers.  This RFC proposes the selection of an external, header-only, modern C++ logging library to be the basis for refactoring the entirety of how iRODS performs logging.  The selected library will allow iRODS to provide structured logging in a distributed manner.  This will increase sysadmins' ability to monitor and track the health and ongoing progress of all iRODS servers under their care.


# Motivation

iRODS logging is currently insufficient and not implemented with an eye towards hundreds of servers in a single zone.  It is currently file-based by default (untested syslog integration notwithstanding) and brittle.  Multiple agents open and close the current log and will occasionally interrupt one another resulting in truncated or interleaved messages.

We are interested in getting the iRODS processes out of the business of writing things down.  We would like to ship the logging activity somewhere else (even if on the same machine) where they can be captured, saved, processed, and potentially analyzed in real-time.  This would also likely speed up the iRODS Agents.

We are also motivated by having less logic in iRODS.  If the very common pattern of logging things is handled by a third-party library, the iRODS source code can be reduced (both potentially speeding things up while reducing the number of bugs).

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

Current research...

https://12factor.net/logs

https://www.loggly.com/blog/structured-logging-then-and-now

https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying

http://bookkeeper.apache.org/distributedlog/

custom log targets

spdlog

https://www.google.com/search?q=c%2B%2B+header+only+logging+library


## Detailed design

What / how.

Outline both "how it works" and "what needs to be changed and in which order to get there."

Describe the overview of the design, and then explain each part of the
implementation in enough detail that reviewers will be able to
identify any missing pieces. Make sure to call out interactions with
other active RFCs.

LOG_NOTICE

Largely, this effort will consist of refactoring the rodsLog() function to use the new library functions.

The new rodsLog() will provide hostname, timestamp, PID.  Additional information to be sent will need to be refactored throughout the codebase to prepare key-value pairs that make sense.

We would like to be able to use this new flavor of logging immediately (as soon as rodsLog() itself is refactored), without requiring a full refactor of every log message within the server.  This means that non-updated rodsLog() calls should be handled cleanly (possibly just creating a default "legacy" key with the value being populated with the existing message).


## Drawbacks

This change is a lot of work to see to completion.  Every logging line will need to be updated.

This change adds a dependency - the new logging library.  This is not much of a risk as we will certainly select an open source library and we will keep a copy of the source ourselves, but it should be mentioned as a source of volatility and/or new bugs.

This change should increase performance, but will probably also increase network traffic from (and potentially to) every iRODS server.  This should be taken into consideration during provisioning as well as monitoring (could skew existing measurements being taken).

This change could break existing monitoring solutions expecting certain outputs and certain patterns.  Those monitors or scripts would need to be refactored/updated to expect the new log format (and location(s)).


## Rationale and Alternatives

Logging is a standard part of any program.  Refactoring the iRODS logging behavior to be in line with other modern network software is necessary and prudent.

The structured logging pattern is robust and has been proven to be both readable for humans and readable/parsable by machines.  Planning to ship the messages over the network as soon as possible allows a distributed system to be monitored and understood remotely (potentially aggregated into a single location).  If/when the system is not behaving as expected, these centralized logs can be monitored giving significantly greater insight into what is happening.

Other possibilities for improving logging has been to incorporate syslog directly/only, rotating the logfiles in a more expected manner, and providing internationalization when writing errors.   A good library may be able to help with all of these issues.

Not providing "modern" logging will continue to hinder iRODS sysadmins' visibility into a complex piece of software that is potentially managing their entire enterprise's infrastructure.  Our logging is better than it has ever been, and it's still not very good (in 4.2).

## Unresolved questions
  
- We need to select a logging library

- We need to consider the number of lines of code that need to be updated.

- What does a best practice look like?

- What does a minimal setup look like?

- What does a very complex setup look like?
