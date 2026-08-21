---
title: SCIM Group Membership Resource Type Extension
abbrev: SCIM Group Membership Resource
docname: draft-zollner-scim-group-members-latest
category: std

ipr: trust200902
area: Applications and Real-Time
workgroup: SCIM
keyword: [Internet-Draft, SCIM]

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]



author:
  -
    name: Danny Zollner
    organization: Okta
    email: danny.zollner@okta.com
  -
    name: Matthias Winter
    organization: Garancy AG
    email: matthias.winter@garancy.com

normative:
  RFC7644:
  RFC7643:
  RFC9865:
---

--- abstract

This document extends the System for Cross-domain Identity Management (SCIM) 2.0 standard by defining a new "GroupMembership" top-level resource. Under the existing model defined in [RFC7643], group memberships are represented as values in a multi-valued attribute within a Group resource. This architecture lacks native support for server-side pagination, filtering, or sorting of individual members. In deployments managing large-scale groups (e.g., 100,000 to 1,000,000 members or more), retrieving a Group resource results in massive HTTP response payloads that can exceed 100MB in size. This can lead to service timeouts, memory exhaustion, and network instability, and has led to many major SCIM implementations choosing to not support returning the value of the "members" attribute for Group resources. This extension introduces a flattened resource model that enables group memberships to benefit from pagination and other SCIM protocol features, ensuring interoperability and performance at scale.

--- middle

# Discussion Venues

This note is to be removed before publishing as an RFC.

Source for this draft and an issue tracker can be found at https://github.com/Zollnerd/scim-group-membership.

# Introduction

The System for Cross-domain Identity Management (SCIM) 2.0 protocol ([RFC7642], [RFC7643], [RFC7644], and [RFC9865]) is widely used for automating the provisioning of identities across disparate systems. While SCIM excels at managing individual User and Group resources, its design for representing relationships, specifically group memberships, encounters significant performance bottlenecks in large-scale enterprise environments.

Currently, the "members" attribute of a Group resource is a multi-valued attribute. Because SCIM only supports paginating resources, a client requesting a Group resource must receive the entire list of group memberships in a single HTTP response. For a group with one million members, an HTTP response can reach approximately 200MB in size. These large payloads create several critical failure points including memory pressure and network timeouts.

This document proposes the "GroupMembership" resource type. By treating a membership as a first-class, top-level resource, service providers can leverage existing SCIM query parameters including filter, count, and multiple pagination methods, allowing them to implement a scalable and reliable interface for managing groups of any size.

# Notational Conventions

{::boilerplate bcp14-tagged}

# The GroupMembership Resource

This section defines the `GroupMembership` resource, which represents a single membership relationship between a SCIM group and a member. By representing each membership as a distinct, top-level resource, service providers can manage group memberships individually, allowing for pagination, filtering, and other operations at scale.

A `GroupMembership` resource represents a direct membership only. Indirect memberships, as defined in [RFC7643] Section 4.1.2, MUST NOT be represented as `GroupMembership` resources.

## Group Membership Base Schema

The base schema for the `GroupMembership` resource type is defined by the URN `urn:ietf:params:scim:schemas:core:2.0:GroupMembership`. The schema defines the following attributes:

group
: A complex attribute that provides a reference to the parent Group resource. Service providers MUST support this attribute and MUST assert that it is always assigned. It MUST be singular and complex. It MUST have the properties `"mutability": "immutable"` and `"required": true` if the service provider supports the creation of Group Membership resources. If creation of Group Membership resources is not supported, the attribute MUST have a `mutability` of `readOnly`. This attribute contains the following sub-attributes:

    value
    : The `id` of the referenced Group resource. Service providers MUST support this attribute and MUST assert that it is always assigned. The sub-attribute value MUST NEVER change. The attribute MUST be singular and of type `string`. It MUST have the same values as the `group` attribute for the properties `mutability` and `required`.

    $ref
    : The URI of the referenced Group resource, as defined in [RFC7643], section 2.4. It MUST be singular and of type `reference`. It MUST have the properties `"referenceTypes": ["Group"]` and "mutability": "readOnly"`.

    display
    : A human-readable name for the referenced Group resource, generally corresponding to the group's `displayName` attribute. The value MUST be the same as that of the `groups.display` sub-attribute of the core User resource if the 'value' sub-attributes references the same Group resource. It MUST be a string. It MUST have the properties `"mutability": "readOnly"` and `"caseExact": false`.
    {: newline="true"}
{: newline="true"}

member
: A complex attribute that provides a reference to the member resource, which can be a User, another Group, or any other resource type that can be a member of a group. Possible resource types are listed in the 'canonicalValues' property of the 'type' sub-attribute and in the 'referenceTypes' property of the '$ref' sub-attribute. This attribute exactly corresponds to the `members` attribute of the core Group resource. Service providers MUST support this attribute and MUST assert that it is always assigned. It MUST be singular and complex. It MUST have the same values for the properties `mutability`, `returned`, and `required` as the `group` attribute. This attribute contains the following sub-attributes:

    value
    : The `id` of the referenced member resource. Service providers MUST support this attribute and MUST assert that it is always assigned. The sub-attribute value MUST NEVER change. The attribute MUST be singular and of type `string`. It MUST have the same values as the `member` attribute for the properties `mutability` and `required`.

    $ref
    : The URI of the referenced Group resource, as defined in [RFC7643], section 2.4. It MUST be singular and of type `reference`. Its `referenceTypes` property MUST reflect the resource types which can be members of the group, e.g.: `["User", Group"]`. It MUST have the property "mutability": "readOnly"`.

    type
    : A string that specifies the resource type of the member, e.g., "User" or "Group". Service provider MUST support this sub-attribute if more than one resource type can be member of a group. The attribute MUST be singular and of type `string`. Its `canonicalValues` property MUST reflect the resource types which can be members of the group, e.g.: `["User", Group"]`. It MUST have the property "mutability": "readOnly"`.

    display
    : A human-readable name for the referenced member resource, generally corresponding to the member's `displayName` attribute. The value MUST be the same as that of the `members.display` sub-attribute of the core Group resource if the 'value' sub-attributes references the same member resource. It MUST be a string. It MUST have the properties `"mutability": "readOnly"` and `"caseExact": false`.
    {: newline="true"}
{: newline="true"}

## JSON Representation

The following is an example of a `GroupMembership` resource in JSON format. This example represents the membership of a user in a group:

~~~
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:GroupMembership"],
  "id": "gm12345",
  "group": {
    "value": "e9e30db",
    "$ref": "https://example.com/scim/v2/Groups/e9e30db",
    "display": "Engineering Team"
  },
  "member": {
    "value": "2819c22",
    "$ref": "https://example.com/scim/v2/Users/2819c22",
    "type": "User",
    "display": "John Doe"
  },
  "meta": {
    "resourceType": "GroupMembership",
    "created": "2026-02-24T20:26:44Z",
    "lastModified": "2026-02-24T20:26:44Z",
    "location": "https://example.com/scim/v2/GroupMemberships/gm12345"
  }
}
~~~

# Managing GroupMembership Resources

This section describes how `GroupMembership` resources are managed using the SCIM protocol. A `GroupMembership` is a simple resource that represents a linkage between a group and a member. As such, a membership can only be created, retrieved, or deleted. Changing the group or the member would fundamentally represent a new membership, not a modification of the existing one. Therefore, a service provider that supports this specification is NOT REQUIRED to support `PATCH` or `PUT` methods for this resource type. However, service providers may extend the resource type with additional attributes and MAY allow clients to change those attributes.

Service providers which support `GroupMembership` resources MUST make all memberships available through the `GroupMembership` resource. If they support management of group memberships (create/delete), they MUST support it through the `GroupMembership` resource.

Service providers MUST ensure that the core User resource's `groups` attribute remains consistent with the state of the `GroupMembership` resources.

Service providers MAY additionally support the `members` attribute of the `Group` resource for retrieval or management of group memberships as defined in [RFC7643] for the purpose of backwards compatibility with existing clients. Service providers that support both the `members` attribute and the `GroupMembership` resource MUST ensure that the state of group memberships remains consistent across both representations. For example, deleting a `GroupMembership` resource MUST result in the corresponding member being removed from the `members` array on the `Group` resource, if that attribute is supported by the service provider.

Service providers MUST ensure that the schema definition for the `Group.members` attribute reflects their support for it. E.g., if the service provider does not support the `Group.members` attribute, it MUST be removed from the schema definition for the `Group` resource. If the service provider does not support retrieving memberships through the `Group.members` attribute, it MUST be marked as `"returned": "never"`. If the service provider does not support adding or removing members through the `Group.members` attribute, it MUST be marked as `"mutability": "readOnly"`.

## Creating GroupMembership Resources (POST)

To add a new member to a group, the client sends a `POST` request to the `GroupMemberships` endpoint. The request body MUST contain a `GroupMembership` resource, specifying the `group.value` and `member.value`.

*   Request: POST /scim/v2/GroupMemberships
*   Response: 201 Created with the full `GroupMembership` resource in the body, including its newly generated `id` and `meta` attributes.

A service provider MUST ensure that both the group and the member referenced by their `id`s exist before creating the `GroupMembership` resource. If either the group or the member does not exist, the service provider SHOULD return a `400 Bad Request` error with a `scimType` of `invalidValue`.

If the membership already exists, the service provider MUST return a `409 Conflict` error.

Example Request Body:

POST /GroupMemberships

~~~
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:GroupMembership"],
  "group": {
    "value": "e9e30db"
  },
  "member": {
    "value": "2819c22"
  }
}
~~~

## Retrieving GroupMembership Resources (GET)

`GroupMembership` resources can be retrieved by sending a `GET` request to the `GroupMemberships` endpoint. Clients can retrieve an individual resource by its `id` or a list of resources.

*   To get a specific membership: `GET /scim/v2/GroupMemberships/{id}`
*   To get all memberships: `GET /scim/v2/GroupMemberships`

### Pagination

Service providers MUST support at least one method of pagination of `GroupMembership` resources to allow clients to retrieve large sets of memberships in manageable chunks.

Index-based pagination is defined in [RFC7644] and uses the `startIndex` and `count` query parameters.

*   Example: `GET /scim/v2/GroupMemberships?startIndex=1&count=1000`

Cursor-based pagination is defined in [RFC9865] and uses the `cursor` and `count` query parameters.

*   Example: `GET /scim/v2/GroupMemberships?count=1000&cursor=aW5kZXg9MTAx`

Clients may let the service provider choose the pagination method by only using the `count` parameter with their first request.

*   Example: `GET /scim/v2/GroupMemberships?count=1000`

The response for a paginated request is a `ListResponse` containing the `GroupMembership` resources for the current page.

### Filtering

Service providers MUST support filtering on the `group.value` and `member.value` attributes. This enables clients to perform critical queries, such as "find all members of a specific group" or "find all groups a specific user is a member of."

To find all members of a group:
: GET /scim/v2/GroupMemberships?filter=group.value eq e9e30db"
{: newline="true"}

To find all groups for a member:
: GET /scim/v2/GroupMemberships?filter=member.value eq "2819c22"
{: newline="true"}

## Deleting GroupMembership Resources (DELETE)

To remove a member from a group, the client sends a `DELETE` request to the URI of the specific `GroupMembership` resource.

*   Request: `DELETE /scim/v2/GroupMemberships/{id}`
*   Response: `204 No Content` on successful deletion.

## Bulk Operations

Clients can create and delete multiple `GroupMembership` resources in a single request using the `/Bulk` endpoint as defined in [RFC7644]. This is highly efficient for synchronizing memberships for a group with many changes.

The following is an example of a `Bulk` request that adds two new members and removes one existing member from a group.

Example Bulk Request:

~~~
{
"schemas": ["urn:ietf:params:scim:api:messages:2.0:BulkRequest"],
"failOnErrors": 1,
"Operations": [
{
  "method": "POST",
  "path": "/GroupMemberships",
  "bulkId": "add-user-1",
  "data": {
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:GroupMembership"],
    "group": { "value": "e9e30db" },
    "member": { "value": "aed987" }
  }
},
{
  "method": "POST",
  "path": "/GroupMemberships",
  "bulkId": "add-user-2",
  "data": {
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:GroupMembership"],
    "group": { "value": "e9e30db" },
    "member": { "value": "bce5231" }
  }
},
{
  "method": "DELETE",
  "path": "/GroupMemberships/gm12345",
  "bulkId": "delete-user-3"
}
]
}
~~~

# Service Provider Considerations

This section describes the requirements for service providers that implement the `GroupMembership` resource.

## Discovering Support for the GroupMembership Resource

Service providers that support the `GroupMembership` resource MUST declare this support in their `/ResourceTypes` and `/Schemas` endpoints.

### ResourceTypes Endpoint

The service provider's `ResourceType` definition, available at the `/ResourceTypes` endpoint, MUST include the resource type definitions for `GroupMembership` with the base schema `urn:ietf:params:scim:schemas:core:2.0:GroupMembership`. The resource type definition MUST correctly reflect the service provider's support, e.g. schema extensions MUST be declared and available at the `/Schemas` endpoint.

**Example ResourceType entry:**

~~~
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:ResourceType"],
  "id": "GroupMembership",
  "name": "GroupMembership",
  "endpoint": "/GroupMemberships",
  "description": "Resource representing a single group membership.",
  "schema": "urn:ietf:params:scim:schemas:core:2.0:GroupMembership",
  "meta": {
    "resourceType": "ResourceType",
    "created": "2026-02-20T13:43:00Z",
    "lastModified": "2026-02-20T13:43:00Z",
    "location": "https://example.com/scim/v2/ResourceTypes/GroupMembership"
  }
}
~~~

### Schema Endpoint

The service provider's `Schema` definition, available at the `/Schemas` endpoint, MUST include the schema definitions for `urn:ietf:params:scim:schemas:core:2.0:GroupMembership` as defined in Section 2.3 of this document. The schema definition MUST correctly reflect the service provider's support, e.g. unsupported sub-attributes MUST be removed from the schema definition.

**Example Schema entry:**

~~~
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Schema"],
  "id": "urn:ietf:params:scim:schemas:core:2.0:GroupMembership",
  "name": "Group Membership",
  "description": "SCIM resource representing a single group membership.",
  "attributes": [
    {
      "name": "group",
      "type": "complex",
      "multiValued": false,
      "mutability": "immutable",
      "returned": "default",
      "required": true,
      "description": "The group of which the member is a member.",
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "mutability": "immutable",
          "returned": "default",
          "required": true,
          "caseExact": true,
          "uniqueness": "none",
          "description": "The id of the group."
        },
        {
          "name": "$ref",
          "type": "reference",
          "referenceTypes": ["Group"],
          "multiValued": false,
          "mutability": "readOnly",
          "returned": "default",
          "required": false,
          "uniqueness": "none",
          "description": "The URI of the group."
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "mutability": "readOnly",
          "returned": "default",
          "required": false,
          "caseExact": false,
          "uniqueness": "none",
          "description": "The displayName of the group."
        }
      ]
    },
    {
      "name": "member",
      "type": "complex",
      "multiValued": false,
      "mutability": "immutable",
      "returned": "default",
      "required": true,
      "description": "The member of the group.",
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "mutability": "immutable",
          "returned": "default",
          "required": true,
          "caseExact": true,
          "uniqueness": "none",
          "description": "The id of the member."
        },
        {
          "name": "$ref",
          "type": "reference",
          "referenceTypes": ["User", "Group"],
          "multiValued": false,
          "mutability": "readOnly",
          "returned": "default",
          "required": false,
          "uniqueness": "none",
          "description": "The URI of the member."
        },
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "mutability": "readOnly",
          "returned": "default",
          "required": false,
          "caseExact": false,
          "uniqueness": "none",
          "description": "The resource type of the member."
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "mutability": "readOnly",
          "returned": "default",
          "required": false,
          "caseExact": false,
          "uniqueness": "none",
          "description": "The displayName of the member."
        }
      ]
    }
  ],
  "meta": {
    "resourceType": "Schema",
    "created": "2026-02-20T13:43:00Z",
    "lastModified": "2026-02-20T13:43:00Z",
    "location": "https://example.com/scim/v2/Schemas/ \
      urn:ietf:params:scim:schemas:core:2.0:GroupMembership"
  }
}
~~~

## Security Considerations

The security considerations for the `GroupMembership` resource are substantially the same as those for the `User` and `Group` resources defined in Section 8 of the SCIM Protocol document [RFC7644]. All requests MUST be made over a secure channel such as Transport Layer Security (TLS).

Authentication and authorization for managing `GroupMembership` resources are the responsibility of the service provider. Implementers should consider the following:

*   A client with permission to read a `Group` resource's `members` attribute MUST be granted permission to `GET` the corresponding `GroupMembership` resources for that group. Access to a `Group` resource alone does not imply access to its member list — a service provider MAY expose group metadata broadly while restricting membership details to privileged clients.

*   A client authorized to add or remove members from a `Group` via a `PATCH` to the `Group` resource MUST have equivalent `POST` and `DELETE` permissions on `GroupMembership` resources for that same group.

## IANA Considerations

This document requests that IANA register the following URNs in the "SCIM Schemas" registry.

**URI:** `urn:ietf:params:scim:schemas:core:2.0:GroupMembership`
**Specification:** This document
**Description:** Defines the schema for a resource representing a single group membership.
