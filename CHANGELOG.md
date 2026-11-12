# Changelog

## [Unreleased]

### Added
- Documented the existing `PATCH /projects/{projectId}` endpoint (partial update; omitted fields keep their current values), with the new `UpdateProject` schema.
- `parentProject` is now marked nullable on create/update: set it to `null` to remove a project from its folder.
- Projects now have a read-only `deleted` field, so deleted projects can be told apart from active ones.
- Documented the existing `status` query parameter of `GET /projects` (`active` or `archived`) to list only active or only archived projects.

## [1.3.0] - 2020-12-11

### Added
- Optional token-based pagination for al list endpoints.
- Endpoint to list all company members.
- Endpoint to list all project members.
- Endpoint to show own member.
- Endpoint to add a member to a project.
- Endpoint to remove a member from a project.
- Migrated documentation to Open API 3.0
