- Feature Name: Metadata inheritance
- Status: draft
- Start Date: YYYY-MM-DD
- Authors: Othmar Weber
- RFC PR: (PR # after acceptance of initial draft)
- iRODS Issue: 

# Summary

The ability to attach metadata to a data object or collection is a core feature of iRODS. Currently the metadata has to be set for each data object seperately. This RFC proposes to introduce a concept of inheriting metadata objects recursively in a file tree. This would ease the metadata handling from user and administrative perspective.

# Motivation

Currently metadata needs to be assigned to each data or collection object individually. In real life you often have the case that certain attributes apply for the whole subtree of a collection. In such cases setting/modifying/deleting metadata always needs a loop on all objects to change the metadata.
By introducing metadata inheritance recursive metadata only needs to be set on a collection level and is inherited automatically the tree down.
This simplifies the metadata handling for recursive attributes from user perspective.

# Guide-level explanation

- introduce the concept of metadata attribute inheritance set on the root of a collection tree 
- provide a means to overwrite the metadata attribute on a subtree  
- provide a means to break the inheritance of an attribute in a subtree 
- the functionality might be implemented on different levels:
1. as inheritance attribute directly attached to the metadata attribute (move from AVU triple to AVUI quadrupel)
2. as additional metadata attribute (e.g. "pass_on_attribute" = "<attribute name>")
3. as recursive option to the imeta command and the APIs to iterate recursively on a collection (not preferred but might have the least impact)
- metadata queries should return the same resultset no matter whether the attribute has been set explicitely or via inheritance

# Reference-level explanation

The implementation depends on how the inheritence is realized:

- additional attribute "I" of the metadata attribute: Requires large set of changes in the area of metadata handling including the data model. Needs also a modification of the query engine. Needs to be attached to only one collection object. Inheritance breaking might be realized easy by setting the same attribute on a lower level. Setting a value means esteblish a new inheritence. Empty value would mean just stop inheriting. 
- as additional metadata attribute: The AVU pass_on_attribute="<attribute name>" would inherit the attribute "<attribute name>" the tree down. Breaking the inheritance could be implemented by just setting the <attribute name>=<empty> on a lower level or define a new pass_on_attribute. Using that logic either needs some magic in the imeta to set the inherited attribute recursively the tree down.
- as recursive option on the imeta command: This would add the option of recursively applying metadata in one run. Once applied the system would handle the metadata like currently. This feature provides the least value since it does not really implement inheritance.

## Detailed design

The detailes highly depend on the way how this feature is implemented:
- additional attribute "I" of the metadata attribute: All areas that deal with metadata need to be checked and changed. New objects that are added to a collection with inheritance enabled need to inherit all recursive attributes. Changing an inherited attribute effects all objects the tree down.
- additional metadata attribute: The attribute "pass_on_attribute" is set on collection level and contains the name of another attribute that exists on the same level and should be inherited the tree downward. If the attribute is empty the inheritence is interrupted. New objects that are added to a collection with inheritance enabled need to inherit all recursive attributes. Changing an inherited attribute effects all objects the tree down.
- recursive option in imeta command: Introduce additional option for the imeta command to allow AVUs to be applied recursively on a complete file tree. 

## Drawbacks

Currently the datamodel is simple. For each object the attributes are stored in a relational table. With inheritance enabled that might change. It might not be easy to keep the queries working transparently. Adding and changing single objects in the catalog might result in cascading updates of metadata. 

## Rationale and Alternatives

Three options with different levels of impact have been proposed.
While just adding a recursive flag to the imeta command has the least impact it does not really provide the feature of metadata inheritance.

Extending the AVU schema to AVUI would need a schema change and effects a lot of code areas but would provide an intrisic feature.

Adding just other metadata to enable inheritence ("```pass_on_attribute```") might have the least impact of all options. It might even be possible to handle that via the rule language. It is not as elegant as the previous options since it just adds some hidden magic to the existing functionality.

## Unresolved questions

Support of the new feature might be implemented first for the icoomands before extending it to the Cloudbrowser or other interfaces.
