# Craftnote API
The Craftnote API allows you to manage data in Craftnote via a HTTP REST API.

## Development
The Craftnote API is currently only available in the Craftnote development environment for testing.

In order to test the API, create an account here https://craftnote-development.firebaseapp.com and send us a short e-mail to th@craftnote.de. We will then create you an API key.

## Swagger Definition
You can explore the API via the included [swagger definition](https://bitbucket.org/Craftnote/api-doc/raw/master/swagger.json). Simply navigate to [editor.swagger.io](https://editor.swagger.io), press File ▸ Import URL and enter the [URL of the swagger definition](https://bitbucket.org/Craftnote/api-doc/raw/master/swagger.json).

## Authentication
You will need to authenticate to the API via the `X-Cn-Api-Key` header. Make sure to include this header on all requests.


## Concepts
### IDs
All entities are identified by uppercase version 4 UUIDs such as:
```
6FEB205E-E5F7-41C5-8BE7-9DBC8DA13998
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
        "parentProject": "46B6D771-492E-4841-8D0C-012328BB1031"
    },
    {
        "id": "46B6D771-492E-4841-8D0C-012328BB1031",
        "name": "MyFolder",
        "creationDate": 1568133915,
        "projectType": "FOLDER",
        "projects": [
            "03AE0AA1-B699-4A9C-85BE-30328897446B"
        ]
    }
  ]
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
**UPCOMING FEATURE**

`PATCH /projects/03AE0AA1-B699-4A9C-85BE-30328897446B`
```json
{
    "name": "NewProjectName"
}
```


## Managing Files
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
            "creationTimestamp": 1568133879
        },
        {
            "id": "7A082960-E44C-4B67-AAB0-294BFB3F5095",
            "name": "myImage.jpg",
            "projectId": "DB5B9A4D-7589-4AF4-8C79-A96A3B796DC9",
            "folderId": null,
            "type": "FOLDER",
            "creationTimestamp": 1568133879
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


## Managing Project Members
**UPCOMING FEATURE**

### List all members of a project
`GET /projects/03AE0AA1-B699-4A9C-85BE-30328897446B/members`
```json
{
  "members": [
    {
        "id": "RlgEBDYUZ6MirS7yOnsNJcDTAjs2",
        "email": "mm@test.de",
        "name": "Max",
        "lastname": "Mustermann",
        "role": "OWNER" 
    }
  ]
}
```

### Add members to a project
`POST /projects/03AE0AA1-B699-4A9C-85BE-30328897446B/members`
```json
{
  "members": [
    {
        "id": "RlgEBDYUZ6MirS7yOnsNJcDTAjs2",
        "role": "EXTERNAL"
    }
  ] 
}
```


### Update member role
`PUT /projects/03AE0AA1-B699-4A9C-85BE-30328897446B/member/RlgEBDYUZ6MirS7yOnsNJcDTAjs2`
```json
{
  "role": "EXTERNAL_SUPERVISOR"
}
```

## Managing Company
**UPCOMING FEATURE**

### List employees

`GET /company/employees`
```json
{
    "employees": [
      {
          "id": "RlgEBDYUZ6MirS7yOnsNJcDTAjs2",
          "email": "mm@test.de",
          "name": "Max",
          "lastname": "Mustermann",
          "role": "OWNER" 
      }
    ]
}
```

### Add employees
`POST /company/employees`
```json
{
    "employees": [
        {
            "email": "mm@test.de",
            "name": "Max",
            "lastname": "Mustermann",
            "role": "OWNER" 
        }
    ]
}
```
