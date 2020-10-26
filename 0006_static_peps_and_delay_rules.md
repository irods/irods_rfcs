- Feature Name: Static PEPs and Delay Rules
- Status: In-Progress
- Start Date: 2020-10-13
- Authors: Kory Draughn
- RFC PR: [#5175](https://github.com/irods/irods/pull/5175)
- iRODS Issue: [#3049](https://github.com/irods/irods/issues/3049), [#4428](https://github.com/irods/irods/issues/4428), [#5153](https://github.com/irods/irods/issues/5153)

# Static PEPs and Delay Rules

## What is this about?
iRODS v4.2.9 contains a lot of changes. Some of the most important and exciting changes have to do with
Intermediate Replicas, Logical Locking, and the Rule Execution (RE) Delay Server. In particular, this thread is
interested in the community's thoughts about the changes to the RE Delay Server.

## What changed?
In previous versions of iRODS, delay rules involved storing the contextual information in a file on the local
hard drive. These files are known as Rule Execution Information files or REI files for short. These files contained
information about who scheduled the rule, auth info, proxy info, data object info, and all sorts of information needed
to successfully execute the rule later. This mostly worked, but presented problems related to performance and hard
drive space. There are also issues with migrating the RE delay server to a different machine.

To get around these issues, iRODS v4.2.9 has moved storage of the contextual information from a file on the local
hard drive to the catalog. The contextual information is stored in a new column (i.e. `r_rule_exec.exe_context`)
as JSON. This solves a multitude of problems. This allows the RE delay server to be migrated without any issues
while also (hopefully) improving performance.

## Are there any downsides to this change?
This should not be viewed as a downside. These changes are a step to future proofing your deployment.

Aside from that, the most important things to know in regards to these changes are:
- Session variables are not allowed in new delay rules. 
- Session variables used in rules prior to upgrade will continue to work, but rely on the REI file. This allows
  administrators to migrate from session variables and Static PEPs to Dynamic PEPs gracefully and at a later time.
- REI files will be migrated into the catalog following execution if and only if the rule does not contain session variables.
- Migrated REI file names will have a suffix of ".migrated_to_catalog" (e.g. rei.<user>.<unique_id>.migrated_to_catalog).

We chose to not add support for session variables due to the following reasons:
- It helps push the community towards Dynamic PEPs.
- It enables a path to deprecating and removing session variables and Static PEPs.
- Session variables and Static PEPs are planned for removal in iRODS v4.3.0.
- Supporting session variables meant we'd have to serialize ALL session variables.

Given those reasons, it did not make sense to add support for them. Especially when the plan is to focus on getting
iRODS v4.3.0 released before UGM 2021.

## Will I have to do anything special to upgrade my deployment?
No. The RE Delay Server will migrate REI files into the catalog following rule execution. This keeps the downtime close
to zero and removes concerns around rules breaking following the upgrade.

## The Community
With all of that said, we are interested in hearing what the community thinks of these changes.
- Q. Do you agree with our decisions?
- Q. How will this affect your deployment?
- Q. Is there a better way to approach this?
- Q. Have we covered all scenarios?

## Related Information
See the following for details on what lead to this solution.
- https://github.com/irods/irods/issues/3049
- https://github.com/irods/irods/issues/4428
- https://github.com/irods/irods/issues/5153
- https://github.com/irods/irods/pull/5175
