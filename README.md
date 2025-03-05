# Craftnote Public API
The Craftnote Public API allows you to manage data in Craftnote via HTTP REST.

## Swagger Definition
You can explore the API via the included [Open API 3.0 definition](https://raw.githubusercontent.com/craftnote/api-doc/master/openapi.yaml). Simply navigate to [editor.swagger.io](https://editor.swagger.io), press File ▸ Import URL and enter the [URL of the Open API definition](https://raw.githubusercontent.com/craftnote/api-doc/master/openapi.yaml).

## Authentication
You will need to authenticate to the API via the `X-CN-API-KEY` header. Make sure to include this header on all requests.
An API Key can be created in the API Keys section of the Craftnote settings by an owner of the company.

In addition to this API key, future versions of the Craftnote Public API will need a client ID that has to be send via the `X-CN-CLIENT-ID` header and can be obtained by contacting Craftnote via [schnittstelle@craftnote.de](mailto:schnittstelle@craftnote.de). 

## Concepts
### IDs
All entities are identified by version 4 UUIDs such as:
```
6FEB205E-E5F7-41C5-8BE7-9DBC8DA13998
```
or other alphanumeric IDs, such as 
```
Ka3gh32ashtQJuuas9
```

### Pagination
Pagination is a way of accessing a list of data in smaller chunks which requires less memory on server and client, and fewer data to be transferred over the network.
The default pagination mode of the Craftnote Public API is an offset-based pagination. Alternatively you can use token based pagination which brings some advantages over the offset-based variant.
In order to understand how each of the pagination modes work let's look at a small example.

Let's consider the following projects:

`GET /projects`
```json
{
  "projects": [
    { "id": "a" },
    { "id": "b" },
    { "id": "c" },
    { "id": "d" },
    { "id": "e" },
    { "id": "f" },
    { "id": "g" },
    { "id": "h" },
    { "id": "i" },
    { "id": "j" },
    { "id": "k" }
  ]
}
```
#### Offset-based
With offset-based pagination you can control the number of items to skip, using the `offset` parameter, and the maximum number of items to return using the `limit` parameter.
So setting `offset` to `2` and `limit` to `5` would yield the following result:

`GET /projects?offset=2&limit=5`
```json
{
  "projects": [
    { "id": "c" },
    { "id": "d" },
    { "id": "e" },
    { "id": "f" },
    { "id": "g" }
  ]
}
```
If offset is omitted, it defaults to `0`. Items are always returned sorted ascending by ID.

#### Token-based
The main issue with offset-based pagination is that if data is added or deleted while querying it can lead to items appearing multiple times, or even not be included at all. A solution to overcome this limitation is token-based pagination.
Token-based pagination needs to be enabled by sending the `paginationMode=token` query parameter. While offset-based pagination simply uses object counts, token-based pagination assumes that each object has a unique ID and uses these IDs to identify the objects to skip.
Setting the query parameters `limit` to `5` and `startAfter` to `c`, we get the following projects:

`GET /projects?paginationMode=token&startAfter=c&limit=5`
```json
{
  "projects": [
    { "id": "d" },
    { "id": "e" },
    { "id": "f" },
    { "id": "g" },
    { "id": "h" }
  ]
}
```
`startAfter` always points to the ID of the last item to skip. If it is omitted, items will be returned from the start (here `id "a"`).  
`limit` works the same way as with offset-based pagination and limits the number of items returned.

Items are always returned sorted ascending by ID.

### Company
The company is the main unit of organization. All projects, members, etc belong to one company.

### Members
A member is the representation of a user in a project or company.
**Example**
```json
{
    "id": "Ka3gh32ashtQJuuas9",
    "email": "cf@gauss.de",
    "mobile": "+49-171-174282137",
    "name": "Carl-Friedrich",
    "lastname": "Gauss"
}
```

### Settings
Settings are defined on a company level and contain general settings like working hours or project statuses.
**Example**
```json
{
 "status": [
  {
   "id": "c6752b29-511f-4bcd-bcea-606f5da2eba9",
   "name": "To Do",
   "description": "This project is waiting to be worked on"
  },
  {
   "id": "890df9a1-12f8-4067-b891-486cb807198d",
   "name": "In Progress",
   "description": "This project is being actively worked on right now"
  },
  {
   "id": "71629908-fc4c-47ce-b1d8-a2423bf56a59",
   "name": "Done",
   "description": "This project has been completed"
  }
 ],
 "workTypes": [
  {
   "id": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
   "name": "Arbeitszeit"
  },
  {
   "name": "Fahrzeit",
   "id": "564cd37a-b3e3-45e1-b6b7-5dcff82cf2a1"
  }
 ]
}
```


### Projects
Craftnote is modelled around the concept of projects. A project has an address, contacts, members.
There are two different types of projects: folder projects and normal (project) projects.
* Normal projects (`projectType: PROJECT`) have a project chat and a file share section.
* Folder projects (`projectType: FOLDER`) can be used to organize projects and can contain other projects or project folders.

Folders keep links to their contained projects with the field `projects` that contains IDs of other projects or folders.

**Example**
```json
{
    "id": "03AE0AA1-B699-4A9C-85BE-30328897446B",
    "name": "MyFirstProject",
    "projectType": "FOLDER",
    "orderNumber": "12312344d",
    "street": "Unter den Linden",
    "zipcode": "01234",
    "city": "Berlin",
    "contacts": [
    {
      "name": "Max Mustermann",
      "emails": ["mm@test.de"],
      "phones": ["+49 711 21893732"]
    }
    ],
    "billingCity": "Berlin",
    "parentProject": "DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9"
}
```
Please refer to the swagger definition for a full list of available fields.

### Files
A project of type `projectType: project` has a file section where users can upload files. Like with projects, files can organized in folders.
Folders are special files with a special type.

File types:
* `type: FOLDER` a folder that can contain other folders or files.
* `type: DOCUMENT` a generic document, such as PDF.
* `type: IMAGE` an image such as a .png or .jpeg file.
* `type: VIDEO` a video.
* `type: AUDIO` an audio message.

The folder of a file is referenced via the `folderId` field.
 
**Example**
```json
{
    "id": "6FEB205E-E5F7-41C5-8BE7-9DBC8DA13998",
    "projectId": "DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9",
    "name": "Projektbericht.pdf",
    "folderId": "7A082960-E44C-4B67-AAB0-294BFB3F5095",
    "type": "DOCUMENT",
    "creationTimestamp": 1567685998,
    "size": 2450
}
```

### Deeplink
A deeplink is a collection of links that can be used to access a specific section from the mobile app or the website.
 **Example**
 ```json
{
  "webLink": "https://app.mycraftnote.de/#/project?id=0DDAAAA9-2508-4AAF-A576-E91E76EA8CDB",
  "appDeepLink": "mycrafty://project?id=0DDAAAA9-2508-4AAF-A576-E91E76EA8CDB"
}
 ```

### Work Time
A work time is used to track time spend on a project per employee. 
**Example**
 ```json
{
  "id": "f048f13c-145c-4e31-ba56-75cb63395af6", 
  "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
  "startTime": 1658736000,
  "endTime": 1658750400,
  "pauseDuration": 2700, 
  "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
  "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
  "userName": "Gauss, Carl-Friedrich",
  "projectId": "c421a59a-3df9-48c1-a1aa-382202aec620",
  "creationTime": 1658839696,
  "state": "stopped"
}
 ```
A work time can have the following states:
* `running` if it has started but not yet ended
* `paused` if it is currently paused
* `stopped` if it has ended
* `deleted` if it has been deleted, i.e. is no longer visible to the user

* The `workTypeId` refers to the `workTypes` that can be accessed from the company settings.

## Webhooks
Webhooks allow you to receive real-time notifications when events occur in your Craftnote account. When an event happens, Craftnote sends an HTTP POST request to the URL you've configured for your webhook.

### Webhook Payload Structure
When an event occurs, Craftnote sends a JSON payload to your webhook URL with the following structure:

```json
{
  "event": "resource.action",
  "timestamp": 1741172456,
  "action": "created",
  "resourceId": "resource-type/resource-id",
  "fieldPath": "$",
  "data": {
    // The complete resource data
  },
  "previousData": null
}
```

- `event`: A string in the format "resource.action" that identifies what happened
- `timestamp`: Unix timestamp (seconds since epoch) when the event occurred
- `action`: The type of action that triggered the event (currently only "created" is supported)
- `resourceId`: A string identifying the resource that was affected, in the format "resource-type/id"
- `fieldPath`: Path to the field that was changed ($ represents the entire resource)
- `data`: The complete current state of the resource
- `previousData`: The previous state of the resource (null for creation events)

### Supported Events
Currently, only creation events are supported for the following resources:

#### Project Events
- `project.created`: Triggered when a new project is created

#### Task Events
- `task.created`: Triggered when a new task is created

#### Chat Message Events
- `message.created`: Triggered when a new chat message is posted

#### Time Tracking Events
- `trackedTime.created`: Triggered when a new time entry is created

#### File Events
- `file.created`: Triggered when a new file is uploaded

### Example Webhook Payload
Here's an example of a webhook payload for a task creation event:

```json
{
  "event": "task.created",
  "timestamp": 1741172456,
  "action": "created",
  "resourceId": "tasks/3db95e59-2be3-45e1-9858-9c56945e526a",
  "fieldPath": "$",
  "data": {
    "id": "3db95e59-2be3-45e1-9858-9c56945e526a",
    "title": "New Task",
    "projectId": "044337df-e9fd-48bc-8f2e-dadd6bc09629",
    "creatorId": "HVU9tHEyejVh8fiBfbyj99zfpjq1",
    "companyId": "19b8ba22-97d0-4e73-96f7-ebeb1b487525",
    "description": "This is a test task",
    "createdTimestamp": 1741172456,
    "assigneeId": "HVU9tHEyejVh8fiBfbyj99zfpjq1",
    "assignee": "McCraftFace, Crafty",
    "lastChangeByUser": "HVU9tHEyejVh8fiBfbyj99zfpjq1",
    "lastEditedTimestamp": 1741172456,
    "isDeleted": false
  },
  "previousData": null
}
```

The structure of the `data` field will match the corresponding resource schema as defined in the API documentation.

## Accessing Company Members
### List all members
`GET /company/members`
```json
{
  "members": [
    {
        "id": "Ka3gh32ashtQJuuas9",
        "email": "cf@gauss.de",
        "mobile": "+49-171-174282137",
        "name": "Carl-Friedrich",
        "lastname": "Gauss"
    }
  ]
}
```
This is a paginated endpoint.

### Show authenticated user
`GET /company/members/me`
```json
{
    "id": "Ka3gh32ashtQJuuas9",
    "email": "cf@gauss.de",
    "mobile": "+49-171-174282137",
    "name": "Carl-Friedrich",
    "lastname": "Gauss"
}
```
This endpoint returns the member that belongs to the provided API key.

## Managing Projects
### List all projects
`GET /projects`
```json
{
  "projects": [
    {
        "id": "03AE0AA1-B699-4A9C-85BE-30328897446B",
        "name": "MyProject",
        "projectType": "FOLDER",
        "orderNumber": "12312344d",
        "street": "Unter den Linden",
        "zipcode": "10234",
        "city": "Berlin",
        "contacts": [
        {
          "name": "Max Mustermann",
          "emails": ["mm@test.de"],
          "phones": ["+49 711 21893732"]
        }
        ],
        "billingCity": "Berlin",
        "parentProject": "46B6D771-492E-4841-8D0C-012328BB1031",
        "archived": false
    },
    {
        "id": "46B6D771-492E-4841-8D0C-012328BB1031",
        "name": "MyFolder",
        "creationDate": 1568133915,
        "projectType": "FOLDER",
        "projects": [
            "03AE0AA1-B699-4A9C-85BE-30328897446B"
        ],
        "archived": true
    }
  ]
}
```
This is a paginated endpoint. The `archived` flag shows if a project is archived for all non-external members.

### Get an individual project
`GET /projects/<projectId>`
```json
{
    "id": "03AE0AA1-B699-4A9C-85BE-30328897446B",
    "name": "MyProject",
    "projectType": "FOLDER",
    "orderNumber": "12312344d",
    "street": "Unter den Linden",
    "zipcode": "10234",
    "city": "Berlin",
    "contacts": [
    {
      "name": "Max Mustermann",
      "emails": ["mm@test.de"],
      "phones": ["+49 711 21893732"]
    }
    ],
    "billingCity": "Berlin",
    "parentProject": "46B6D771-492E-4841-8D0C-012328BB1031",
    "archived": false
}
```

### Get a project deeplink
`GET /projects/<projectId>/deeplink`
```json
{
  "webLink": "https://app.mycraftnote.de/#/project?id=0DDAAAA9-2508-4AAF-A576-E91E76EA8CDB",
  "appDeepLink": "mycrafty://project?id=0DDAAAA9-2508-4AAF-A576-E91E76EA8CDB"
}
```

### Create new project
`POST /projects`
```json
{
    "name": "MyProject",
    "projectType": "FOLDER",
    "orderNumber": "12312344d",
    "street": "Unter den Linden",
    "zipcode": "10234",
    "city": "Berlin",
    "contacts": [
    {
      "name": "Max Mustermann",
      "emails": ["mm@test.de"],
      "phones": ["+49 711 21893732"]
    }
    ],
    "billingCity": "Berlin",
    "parentProject": "46B6D771-492E-4841-8D0C-012328BB1031"
}
```

### Replace project
`PUT /projects/<projectId>`
```json
{
    "name": "My Changed name",
    "projectType": "FOLDER",
    "orderNumber": "12312344d",
    "street": "Unter den Linden",
    "zipcode": "10234",
    "city": "Berlin",
    "contacts": [
    {
      "name": "Max Mustermann",
      "emails": ["mm@test.de"],
      "phones": ["+49 711 21893732"]
    }
    ],
    "billingCity": "Berlin",
    "parentProject": "46B6D771-492E-4841-8D0C-012328BB1031",
    "statusId": "e3f98abb-09f5-434c-af41-42d1b429babd"
}
```

You can archive projects for all non-external members by setting the `archived` flag in an update request:
```
...
"archived": true
...
```

### Update project
`PATCH /projects/<projectId>`
```json
{
    "name": "My Changed name"
}
```

You can archive projects for all non-external members by setting the `archived` flag in an update request:
```
...
"archived": true
...
```


## Accessing Project Members
### List all members
`GET /projects/<projectId>/members`
```json
{
  "members": [
    {
        "id": "Ka3gh32ashtQJuuas9",
        "email": "cf@gauss.de",
        "mobile": "+49-171-174282137",
        "name": "Carl-Friedrich",
        "lastname": "Gauss"
    }
  ]
}
```
This is a paginated endpoint.

### Add a member
`POST /projects/<projectId>/members`
```json
{
  "userId": "Ka3gh32ashtQJuuas9"
}
```

### Remove a member
`DELETE /projects/<projectId>/members/Ka3gh32ashtQJuuas9`

## Managing Project Files
INFO: This manages project files. Please be aware that a project also has a chat section which can have files. These files are not managed with the following endpoints.

### List all files of a project
`GET /projects/<projectId>/files`
```json
{
    "files": [
        {
            "id": "7811C21D-AC17-479F-9C2D-D6AC6AF0F1C0",
            "name": "myImage.jpg",
            "projectId": "DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9",
            "folderId": "7A082960-E44C-4B67-AAB0-294BFB3F5095",
            "type": "IMAGE",
            "creationTimestamp": 1568133879,
            "lastModifiedTimestamp": 1568133879
        },
        {
            "id": "7A082960-E44C-4B67-AAB0-294BFB3F5095",
            "name": "myImage.jpg",
            "projectId": "DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9",
            "folderId": null,
            "type": "FOLDER",
            "creationTimestamp": 1568133879,
            "lastModifiedTimestamp": 1568133879
        }
    ]
}
```
This is a paginated endpoint.

### Get individual file
`GET /files/<fileId>`
```json
{
    "id": "7811C21D-AC17-479F-9C2D-D6AC6AF0F1C0",
    "name": "myImage.jpg",
    "projectId": "DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9",
    "folderId": "7A082960-E44C-4B67-AAB0-294BFB3F5095",
    "type": "IMAGE",
    "creationTimestamp": 1568133879
}
```

### Download file
`GET /files/<fileId>?download=true`
```
...
```

### Upload file
```
POST /projects/<projectId>/files
type=image
name=myImage
file=<image.png>
```

### Update file
```
PUT /files/<fileId>
type=image
file=<image.png>
```

### Setting file permissions
Permissions can be on root level folders set during creation or when updating a file folder
```
POST /files/<fileId>
type=folder
name=Folder
permission=EMPLOYEE
permission=EXTERNAL
```

```
PUT /files/<fileId>
type=folder
name=Folder
permission=EMPLOYEE
permission=EXTERNAL
```

You can set permissions for `EMPLOYEE`, `EXTERNAL`, `SUPERVISOR`, `EXTERNAL_SUPERVISOR` and `OWNER`.



## Managing Work Times
### List all work times of company
`GET /times`
```json
{
 "times": [
  {
   "id": "3153b5e7-1bf5-4109-99f9-f6576b1d63c9",
   "creationTime": 1663336442,
   "startTime": 1662016020,
   "endTime": 1662044400,
   "pauseDuration": 3600,
   "projectId": "79ed902a-2e61-4e38-b4f9-f9d5ae14f016",
   "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
   "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
   "userName": "Gauss, Carl-Friedrich",
   "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
   "state": "stopped"
  },
  {
   "id": "7667a828-0664-40a8-9136-1daf1048f34b",
   "creationTime": 1663336443,
   "startTime": 1662016020,
   "endTime": 1662044400,
   "pauseDuration": 3600,
   "projectId": "79ed902a-2e61-4e38-b4f9-f9d5ae14f016",
   "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
   "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
   "userName": "Gauss, Carl-Friedrich",
   "state": "stopped",
   "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1"
  },
  {
   "id": "10f0bcd6-3ce0-4efc-98ac-ddbbe5bca9ad",
   "creationTime": 1663077341,
   "startTime": 1662016020,
   "endTime": 1662644921,
   "pauseDuration": 0,
   "projectId": "9273deee-ebbb-4185-a321-e0fa597dd512",
   "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
   "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
   "userName": "Gauss, Carl-Friedrich",
   "state": "deleted",
   "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1"
  }
 ]
}
```
This is a paginated endpoint. It lists all work times of the entire company.

### List all work times of project
`GET /projects/<projectId>/times`
```json
{
 "times": [
  {
   "id": "3153b5e7-1bf5-4109-99f9-f6576b1d63c9",
   "creationTime": 1663336442,
   "startTime": 1662016020,
   "endTime": 1662044400,
   "pauseDuration": 3600,
   "projectId": "79ed902a-2e61-4e38-b4f9-f9d5ae14f016",
   "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
   "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
   "userName": "Gauss, Carl-Friedrich",
   "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
   "state": "stopped"
  },
  {
   "id": "7667a828-0664-40a8-9136-1daf1048f34b",
   "creationTime": 1663336443,
   "startTime": 1662016020,
   "endTime": 1662044400,
   "pauseDuration": 3600,
   "projectId": "79ed902a-2e61-4e38-b4f9-f9d5ae14f016",
   "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
   "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
   "userName": "Gauss, Carl-Friedrich",
   "state": "stopped",
   "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1"
  }
 ]
}
```
This is a paginated endpoint. It lists only work times of the chosen project.

### Get individual work time
`GET /projects/<projectId>/times/<timeId>`
```json
  {
 "id": "3153b5e7-1bf5-4109-99f9-f6576b1d63c9",
 "creationTime": 1663336442,
 "startTime": 1662016020,
 "endTime": 1662044400,
 "pauseDuration": 3600,
 "projectId": "79ed902a-2e61-4e38-b4f9-f9d5ae14f016",
 "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
 "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
 "userName": "Gauss, Carl-Friedrich",
 "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
 "state": "stopped"
}
```

### Create work time
`POST /projects/<projectId>/times`
```json
  {
 "id": "3153b5e7-1bf5-4109-99f9-f6576b1d63c9",
 "creationTime": 1663336442,
 "startTime": 1662016020,
 "endTime": 1662044400,
 "pauseDuration": 3600,
 "projectId": "79ed902a-2e61-4e38-b4f9-f9d5ae14f016",
 "workTypeId": "0eec4463-72c0-4311-bc4a-04f79c5a2d95",
 "userId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
 "userName": "Gauss, Carl-Friedrich",
 "creatorId": "9GXvGejHIMU4yv1fGu1wUxMZjLz1",
 "state": "stopped"
}
```
