- Feature Name: Standardize Metadata Attribute Names For Common Filesystem Properties
- Status: draft
- Start Date: 2018-12-06
- Authors: Kory Draughn
- RFC PR: (PR # after acceptance of initial draft)
- iRODS Issue: (one or more # from the issue tracker)

# Summary

Many, if not all users will need to attach metadata to the objects within iRODS. It is likely that this 
metadata will be filesystem related information (e.g. uid, gid, mtime). This RFC proposes that metadata 
attribute names for common filesystem properties be standardized. Having standard names would ease 
development of software and allow users to take advantage of tools developed by others.

# Motivation

Currently, iRODS does not provide any conventions for naming metadata attributes. This is mostly okay. 
This breaks down when people want to release software that could benefit others, but relies on a particular 
metadata attribute naming scheme. For example:
```python
 1 # /var/lib/irods/stat_eventhandler.py
 2 from irods_capability_automated_ingest.core import Core
 3 from irods_capability_automated_ingest.utils import Operation
 4 from irods.meta import iRODSMeta
 5 import subprocess
 6 
 7 class event_handler(Core):
 8     @staticmethod
 9     def to_resource(session, meta, **options):
10          return "targetResc"
11
12      @staticmethod
13      def operation(session, meta, **options):
14          return Operation.PUT
15
16      @staticmethod
17      def post_data_obj_create(hdlr_mod, logger, session, meta, **options):
18          args = ['stat', '--printf', '%04a,%U,%u,%G,%g,%x,%y', meta['path']]
19          out, err = subprocess.Popen(args, stdout=subprocess.PIPE, stderr=subprocess.PIPE).communicate()
20          s = str(out.decode('UTF-8')).split(',')
21          print(s)
22          obj = session.data_objects.get(meta['target'])
23          obj.metadata.add("filesystem::perms", s[0], '')
24          obj.metadata.add("filesystem::owner", s[1], '')
25          obj.metadata.add("filesystem::uid",   s[2], '')
26          obj.metadata.add("filesystem::group", s[3], '')
27          obj.metadata.add("filesystem::gid",   s[4], '')
28          obj.metadata.add("filesystem::atime", s[5], '')
29          obj.metadata.add("filesystem::mtime", s[6], '')
30          session.cleanup()
```
This python script is an example of how someone might attach metadata to multiple objects using Automated 
Ingest. From line 23 to line 29, several pieces of file information, such as permissions and group id, are 
harvested and stored as metadata. The metadata attribute names use the following scheme: `filesystem::<property>`. 
This works for systems that understand that scheme, but it's likely there aren't many. Without standardization, 
users **WILL** invent their own names. It would be much better if all systems could use common metadata attribute 
names.

### Which Filesystem Properties Should Be Standardized?

Below are some of the most heavily used properties of any filesystem. The majority of these properties can be 
found in many operating systems. Because iRODS provides a virtual filesystem (following the POSIX standard), it 
should also provide standard metadata attribute names for these properties, just like other filesystems.

- Permissions
- Owner Name
- Owner ID
- Group Name
- Group ID
- Create Time
- Access Time
- Modify Time

### Possible Schemes For Standard Metadata Attribute Names

The schemes below are only suggestions. The pros and cons of these have not been investigated. They are here 
to serve as a starting point for the design.

- irods::filesystem::\<property\>
- irods::fsys::\<property\>
- irods::fs::\<property\>
- irods::\<property\>
- filesystem::\<property\>
- fsys::\<property\>
- fs::\<property\>

