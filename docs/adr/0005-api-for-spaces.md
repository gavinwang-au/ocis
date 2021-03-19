# 5. Graph API for Spaces

* Status: proposed
* Deciders: @butonic, @micbar, @dragotin, @hodyroff, @pmaier1
* Date: 2021-03-19

Technical Story: API to enable the concept of [Spaces](https://github.com/owncloud/enterprise/issues/3863)

## Context and Problem Statement

As one of the building blocks for Spaces in oCIS we plan to add an API that returns information about available spaces.

> Note: The term "spaces" is used here in the context of "a space where files can be saved", similar to a directory. It is not to be confused with space in the sense of file space for example.

The purpose of this new API is to give clients a very simple way to query the dynamic list of spaces, that the user has access to. Clients can provide a way better user experience with that.

This API would even allow to provide (WebDAV-) endpoints depending on the kind and version of the client asking for it.

This ADR discusses how this API could be designed.

## Decision Drivers

- Make it easy to work with a dynamic list of spaces of a user for the clients.
- No longer the need to make assumptions about WebDAV- and other routes in clients.
- More meta data available about spaces for a better user experience.
- Part of the bigger spaces plan.
- Important to consider in client migration scenarios, ie. in CERN.

## Considered Options

1. [Microsoft Graph API](https://developer.microsoft.com/en-us/graph) inspired API that provides the requested information.

## Decision Outcome

This the DRAFT for the API.

### API to Get Info about Spaces

ownCloud servers provide an API to query for available spaces of an user.

Most important, the API returns the WebDAV endpoint for each space. With that, clients do not have to make assumptions about WebDAV routes any more.

See [Drive item in Microsoft Graph API](https://docs.microsoft.com/de-de/graph/api/resources/onedrive?view=graph-rest-1.0)

### Get "Home folder"

Retrieve information about the home space of a user. Note: This is only a subset of the spaces the user might have access to.

API Call: `/me/drive`: Returns the information of the users home folder.

### Get All Spaces of a User

Retrieve a list of available spaces of a user. This includes all spaces the user has access to at that moment, also the home space.

API Call: `/me/drives`: Returns a list of spaces.

### Common Reply

The reply to both calls is either one or a list of [Drive representation objects](https://docs.microsoft.com/de-de/graph/api/resources/drive?view=graph-rest-1.0):

```
{
  "id": "string",
  "createdDateTime": "string (timestamp)",
  "description": "string",
  "driveType": "personal | business | documentLibrary",
  "eTag": "string",
  "lastModifiedDateTime": "string (timestamp)",
  "name": "string",
  "owner": { "@odata.type": "microsoft.graph.identitySet" },
  "webUrl": "url"
}

```

The meaning of the object are in ownClouds context:

1. **id** - a unique ID identifying the space
2. **driveType** - describing the type of the space.
3. **eTag** - the current ETag of the space
4. **owner** - an owner object to whom the space belong
5. **webUrl** - the relative Webdav path in ownCloud. It must include the `remote.php` part in oC10 installations.


The following driveType values are available in the first step, but might be enhanced later:

* **personal**: The users home space
* **documentLibrary**: The project spaces available for the user
* **shares**: The share jail, contains all shares for the user (*)

The (*) marked types are not defined in the official MS API.

### Positive Consequences

- A well understood and mature API from Microsoft adopted to our needs.
- Prerequisite for Spaces in oCIS.
- Enables further steps in client development.

### Negative Consequences

- Migration impact on existing installations. Still to be investigated.

### Open Topics

- Do we need a new WebDAV endpoint?
- What are the WebDAV pathes for Trashbin, Versions
	+ option: additional entries in the reply struct
