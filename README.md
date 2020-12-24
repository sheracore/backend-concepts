# backend-concepts

# Django
## Permissions
Permissions are used to grant or deny access for different classes of users to different parts of the API.
Before running the main body of the view each permission in the list is checked. If any permission check fails an exceptions.PermissionDenied or exceptions.NotAuthenticated exception will be raised, and the main body of the view will not run.
