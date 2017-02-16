# API Versioning guide
This guide describes when and how you can version REST APIs.

## Versioning model
We choose to follow semantic versioning for our API. In short this means the following:

 - Breaking changes that require intervention of the user: increase major version
 - Changes that add functionality without requiring manual intervention: increase minor version
 - Things that don't change the interface: increase patch version
 
It is important to keep this model in mind when reading the rest of the guide so you get a 
good idea of what we are trying to achieve with our versioning strategy.

## Versioning the API
There are many ways to version the API. We choose to use a strategy where we give each version
of the API a separate URL that looks like this: `/{api-endpoint}/{version}`. This means
you that the API will have a URL `/products/v1` and later when a breaking change is implemented `/products/v2`.

We choose to use separate URLs and consequently deployments, because this is by far the easiest to work with.
For a new major version we deploy a new copy of the app on a new URL. We leave the old API alone and only patch it when necessary.
This way we don't pull versioning into our code. So we can focus on the business problem instead of building boilerplate code.

There are other ways to version your API. You can use a HTTP header to indicate the version to use. Or you could specify which version 
you want by sending an Accept Header that looks like this: `application/vnd.api.v2+json`. But both of these techniques are more 
complicated and we don't want that at this stage.

In the future we could allow these version schemes by using an API gateway that is capable of routing requests to the right instances of our API based on these headers. But for now we choose to use the URL approach.

The driving principle behind our versioning is that we explicitly choose not to version the API within the code of the API solution. We think of an API version as the version of a piece of software. Not some variant of a class within the API. This should be represented in our code.

## When to version the API
With the versioning model in mind you can argue that we need to change the URL when we change something in the API.
We choose not to version for every small change. We only change the URL when there's a breaking change.

Minor and Patch changes don't have impact on existing consumers. So it would only make things harder for them if we change 
the url for those types of changes. 

In practice this means we increase the version of the API when one of the following events happen:

 - We add fields to the data schema of the API that are required.
 - We move fields within the schema of the API.
 - We remove fields from the API and raise errors when someone uses them anyway.
 - We change URLs of resources in our API.

We choose not to version the API when one of the following events happn:

 - We add new optional fields to the schema of the API.
 - We add new resources to the API.
 - We fix bugs in the behavior of the API.
 
## Maintaining old versions of the API
Versioning the API means we have old copies that become deprecated. We need to make sure that we don't get overburdened by the 
old versions of the API. Because of that we choose a strategy that allows us to keep the amount of maintained versions to a minimum.

In principle, we only maintain the latest version. So if we deploy `v2` it means that `v1` is deprecated and should not be used.
We want to motivate our consumers to move to the latest version. But we only remove the old version when it is no longer used.

When we deploy a new version of the API, we notify customers that they have three weeks to move over to the new version of the API. This should be enough for most. If needed we can extend this period, but this needs to be discussed with the team first so we know what customer is having trouble to upgrade.

To monitor the usage of our API we use a tool like `Application Insights` to keep track of the requests sent to our API.
When the previous version no longer receives requests it is removed from the web server. Using a monitoring tool also allows
us to actively motivate consumers to move to the new version.
