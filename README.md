# backend-concepts
#########################
####### sheracoer #######
#########################

* You can map CRUD methods in your terminla for example:
```
http post http://127.0.0.1:8000/api-token-auth/ username=vitor password=123
```

# Django
## Django rest framework
###  Permissions
Permissions are used to grant or deny access for different classes of users to different parts of the API.
Before running the main body of the view each permission in the list is checked. If any permission check fails an exceptions.PermissionDenied or exceptions.NotAuthenticated exception will be raised, and the main body of the view will not run.
When the permissions checks fail either a "403 Forbidden" or a "401 Unauthorized" response will be returned, according to the following rules:

### Authentication
Authentication is always run at the very start of the view, before the permission and throttling checks occur, and before any other code is allowed to proceed.
The request.user property will typically be set to an instance of the contrib.auth package's User class.
The request.auth property is used for any additional authentication information, for example, it may be used to represent an authentication token that the request was signed with.
Note: Don't forget that authentication by itself won't allow or disallow an incoming request, it simply identifies the credentials that the request was made with.

#### How authentication is determined
The authentication schemes are always defined as a list of classes. REST framework will attempt to authenticate with each class in the list, and will set request.user and request.auth using the return value of the first class that successfully authenticates.
If no class authenticates, request.user will be set to an instance of django.contrib.auth.models.AnonymousUser, and request.auth will be set to None.
The value of request.user and request.auth for unauthenticated requests can be modified using the UNAUTHENTICATED_USER and UNAUTHENTICATED_TOKEN settings.
#### You can also set the authentication scheme on a per-view or per-viewset basis, using the APIView class-based views.
```
from rest_framework.authentication import SessionAuthentication, BasicAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    authentication_classes = [SessionAuthentication, BasicAuthentication]
    permission_classes = [IsAuthenticated]

    def get(self, request, format=None):
        content = {
            'user': unicode(request.user),  # `django.contrib.auth.User` instance.
            'auth': unicode(request.auth),  # None
        }
        return Response(content)
```
Or, if you're using the @api_view decorator with function based views.
```
@api_view(['GET'])
@authentication_classes([SessionAuthentication, BasicAuthentication])
@permission_classes([IsAuthenticated])
def example_view(request, format=None):
    content = {
        'user': unicode(request.user),  # `django.contrib.auth.User` instance.
        'auth': unicode(request.auth),  # None
    }
    return Response(content)
```
