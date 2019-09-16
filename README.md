# Craftnote API
The Craftnote API allows you to manage data in Craftnote via a HTTP REST API.

## Development
The Craftnote API is currently only available in the Craftnote development environment for testing.

In order to test the API, create an account here https://craftnote-development.firebaseapp.com and send us a short e-mail to th@craftnote.de. We will then create you an API key.

## Swagger Definition
You can explore the API via the included [swagger definition](swagger.json). Simply navigate to [editor.swagger.io](https://editor.swagger.io), press File ▸ Import URL and enter the [URL of the swagger definition](swagger.json).

## Authentication
You will need to authenticate to the API via the `X-Cn-Api-Key` header. Make sure to include this header on all requests.


## Concepts
### IDs
All entities are identified by version 4 UUIDs such as:
```
0769cf8e-cda6-4565-ac76-cd68912d0e33
```

### Projects
Craftnote is modelled around the concept of projects. A project has an address, contacts, members.
There are two different types of projects: folder projects and normal (project) projects.
* Normal projects (`projectType: project`) have a project chat and a file share section.
* Folder projects (`projectType: project`) can be used to organize projects and can contain other projects or project folders.

Folders keep links to their contained projects with the field `projects` that contains IDs of other projects or folders.

```

```

### Files
A project of type `projectType: project` has a file section where users can upload files. 


 


## Managing Projects

You can list all available projects and projects

## Managing Files

## Managing Members

