# API styleguide
Based on previous experiences with API development at Info Support we wanted
to document how we think APIs should be build. This document provides general
guidance on how to build APIs and is loosly based on a guide published by
paypal.

## URI components
We have but a few general rules to identify resources in our API. The general
idea behind these rules is that our APIs are resource oriented whenever we can.

### Resource identification
To access a resource within the API you need to know the name of the resource
and its identifier. We use business identifiers as much as possible to make the
API easier to use.

We specify resource identifiers like so:

```
/{namespace}/{version}/{resource}/{resource-id}/{sub-resource}/{sub-resource-id}
```

Note that each resource is plural in our API. Unless we are absolutely sure that
there's only one of the resource ever. This doesn't happen very often, but it
can happen. For example, namespaces in the API aren't pluralized.

### RPC endpoints
In general we frown upon endpoints that are RPC oriented. Things like
`/aanvragen/1/finalize` are a bad practice. But sometimes there is no other way
to provide access to functionality in the API. Simply because it's a specific
action that you need to perform on a resource.

We follow this three step plan to provide a spot for RPC oriented endpoints:

 - Check if it isn't actually a new resource (noun)
 - Check if you can provide another way to solve the problem, for example by
   providing specific fields on an existing endpoint
 - If you still haven't found a spot, pick the closest related resource and
   add the action to that resource using the template provided below.

RPC style endpoints are always provided using the following template:

```
POST /.../{resource}/{resource-id}/actions/{action}
```

This way the users of our API know what to expect and what they are actually
doing with the API.

**Note:** All action oriented resources use a POST method for the request.

RPC style endpoints have a benefit over resource style endpoints for complex
operations since we want to be absolutely sure that the user of our API
understands what he/she is doing.

The downside to RPC style endpoints is that they are less usable. URLs for RPC
style endpoints are opaque, you can't get any information about the resource
structure from them. We have tried to solve this using a good URL pattern, but
the RPC style endpoints should still be used with caution.

## Collection resources
Most of our resources are collections of things. To make sure that the users of
our API know how to use the interface we provide a set of rules to make
things easy.

### URL template
```
GET /.../{resource}[?page_index={page_index}]
```

A resource collection can be retrieved using the `GET` verb. Please note that
getting a collection resource does not change it.

All items in the collection resource are provided through the `items` property.
This separates the items from other metadata provided by the collection
resource.

### Paging
If a collection resource is supposed to support paging it should have additional
metadata:

 - total_items: the number of items in the collection
 - page_size: the number of items per page
 - page_index: the page index retrieved

You should be able to provide a `page_index` parameter on the query string in
order to retrieve a specific page from the collection resource.

### Sub-collections
Although you can create collections with unlimited depth. We put a limit on the
number of levels in resource collections to 2. This ensures that the URL is of a managable size.

When you apply the rule of max 2 layers of sub resources you end up with a resource 
collection pattern at maximum depth that looks like this:

```
/{resource}/{resource-id}/{sub-resource}/{sub-resource-id}
```

### Example request:

```
GET /aanvragen?page_index=0
```

Response:

```
200 OK

{
    "total_items": 112,
    "page_size": 20,
    "page_index": 0,
    "items:" [
        ....
    ]
}
```

### Status codes
 - 200 OK: Resource collection available, regardless of number of items in
   the collection.
 - 404 NOT FOUND: Resource collection doesn't exist.

## Read single resource
As part of a collection it is possible to request a single resource from our API.
Resources are usually identified by a business property like transaction number.
Technical IDs in the URL should be avoided if possible.

### URL template
```
/{resource}/{resource-id}
```

### Example request
```
GET /aanvragen/123
```

Response:

```
200 OK
{
  "status": "InBehandeling"
}
```

In contrast to collection resources, you will receive just the data for a
single item. There's no additional metadata that you need to specify.

### Status codes
 - 200 OK: Resource available for retrieval
 - 404 NOT FOUND: Resource doesn't exist

## Update single resource
To update a single resource you can use the `PUT` verb. Please note that the
shape of the PUT request should match that of the GET response.
This makes the API much more usable for clients.

**Note:** If you don't use a field from the `GET` request, make sure that you
ignore it in the `PUT`. Don't send a `400 BAD REQUEST` when you get fields
that you didn't expect.

### URL template
```
PUT /{resource}/{resource-id}
{
  "status": "InBehandeling",
  "aanvrager": {

  }
}
```

## Example request
```
PUT /aanvragen/123
```

Response

```
200 OK

{
    ...
}
```

Notice that a PUT operation returns the resource after the request is completed.
The response is the same as with a regular GET operation.

### Status codes
 - 200 OK: Resource found and updated
 - 404 NOT FOUND: Resource not found
 - 400 BAD REQUEST: The update failed, because of a validation error

## Create a new resource
Creates a new item in a collection within the API. The request body for this kind
of request is different from the update operation in that we sometimes accept
less values in a create operation.

### URL template
```
POST /{resource}
```

### Example request
```
POST /aanvragen
```

Response:

```
201 CREATED

{
    ...
}
```

The response body for a create operation is the same as for a `GET` operation.
The operation should return the resource created by the operation.

### Status codes
 - 201 CREATED: The resource was succesfully stored
 - 400 BAD REQUEST: There was a validation error

## Delete a resource
Deletes an existing item from the resource collection. This operation is not
idempotent. You cannot delete the same item twice.

### URL template
```
DELETE /{resource}/{resource-id}
```

### Example request
```
DELETE /aanvragen/123
```

Response:

```
204 NO CONTENT
```

### Status codes
 - 204 NO CONTENT - The resource was deleted
 - 404 NOT FOUND - The resource was not found
