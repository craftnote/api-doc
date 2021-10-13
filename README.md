# Craftnote Public API
The Craftnote Public API allows you to manage data in Craftnote via HTTP REST.

## Swagger Definition
You can explore the API via the included [Open API 3.0 definition](https://bitbucket.org/Craftnote/api-doc/raw/master/openapi.yaml). Simply navigate to [editor.swagger.io](https://editor.swagger.io), press File ▸ Import URL and enter the [URL of the Open API definition](https://bitbucket.org/Craftnote/api-doc/raw/master/openapi.yaml).

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
With offset-based pagination you can control the number of items to skip, using the `offset` parameter, and the maxmimum number of items to return using the `limit` parameter.
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
The `archived` flag shows if a project is archived for all non-external members.

### Get an individual project
`GET /projects/03AE0AA1-B699-4A9C-85BE-30328897446B`
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
`GET /projects/03AE0AA1-B699-4A9C-85BE-30328897446B/deeplink`
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

### Update project
`PUT /projects/03AE0AA1-B699-4A9C-85BE-30328897446B`
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
    "parentProject": "46B6D771-492E-4841-8D0C-012328BB1031"
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
`GET /projects/03AE0AA1-B699-4A9C-85BE-30328897446B/members`
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

### Add a member
`POST /projects/03AE0AA1-B699-4A9C-85BE-30328897446B/members`
```json
{
  "userId": "Ka3gh32ashtQJuuas9"
}
```

### Remove a member
`DELETE /projects/03AE0AA1-B699-4A9C-85BE-30328897446B/members/Ka3gh32ashtQJuuas9`

## Managing Project Files
INFO: This manages project files. Please be aware that a project also has a chat section which can have files. These files are not managed with the followin endpoints.

### List all files of a project
`GET /projects/DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9/files`
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
            "lastModifiedTimestamp": 1568133879,
        },
        {
            "id": "7A082960-E44C-4B67-AAB0-294BFB3F5095",
            "name": "myImage.jpg",
            "projectId": "DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9",
            "folderId": null,
            "type": "FOLDER",
            "creationTimestamp": 1568133879,
            "lastModifiedTimestamp": 1568133879,
        }
    ]
}
```

### Get individual file
`GET /files/7811C21D-AC17-479F-9C2D-D6AC6AF0F1C0`
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
`GET /files/7811C21D-AC17-479F-9C2D-D6AC6AF0F1C0?download=true`
```
...
```

### Upload file
```
POST /projects/DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9/files
type=image
name=myImage
file=<image.png>
```

### Update file
```
PUT /files/7811C21D-AC17-479F-9C2D-D6AC6AF0F1C0
type=image
file=<image.png>
```
