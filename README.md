# I Neve Ever Give Up
## backend-concepts
#########################
####### sheracoer #######
#########################

* You can map CRUD methods in your terminla for example:
```
self.user = create_user(email='private@sheracore.com',password='testpass',name='name')
http post http://127.0.0.1:8000/api-token-auth/ username=vitor password=123
```
# Django
## Django rest framework
### TestCase (TDD)
After patch or put to update user or any other models you must refres db
```
self.user.refresh_from_db()
```

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
#### How to create token
To authenticate users you should create token for each users when thay login and in other requestes by that users thay are autenticated.
in django rest framework ObtainAuthToken do this, if your user customized so you should use ObtainAuthToken to create a view for creating token by email and password(custom user) instead of username and password.
```
from rest_framework import permissions, generics
from rest_framework.authtoken.views import ObtainAuthToken
from rest_framework.settings import api_settings

from user.serializers import UserSerializer, AuthTokenSerializer


class CreateUserView(generics.CreateAPIView):
	"""Create a new user in the system"""
	serializer_class = UserSerializer


class CreateTokenView(ObtainAuthToken):
	"""Create a new auth token for user"""
	serializer_class = AuthTokenSerializer
	renderred_classes = api_settings.DEFAULT_RENDERER_CLASSES

```
To create it's serializer(AuthTokenSerializer) you shuld use Serializer instead of serializer beacuse you must authenticate email and password by authenticate class. 
By passing the username(email in here) and password to the authenticate you can authenticate a request
```
from rest_framework import serializers

from django.contrib.auth import get_user_model, authenticate
# For outputing any text to the screen its good idea use this translation tool(gettext_lazy)
from django.utils.translation import gettext_lazy as _

class AuthTokenSerializer(serializers.Serializer):
	"""Serializer for the user authentication object"""
	email = serializers.CharField()
	password = serializers.CharField(
		style={'input_type': 'password'},
		trim_whitespace=False
		)

	# attrs equals is evry fields that make up our serializers (email,password)
	def validate(self, attrs):
		"""Validate and authenticate the user"""
		email = attrs.get('email')
		password = attrs.get('password')
		
		user = authenticate(
			request = self.context.get('request'),
			username=email,
			password=password
			)
		if not user:
			msg = _('Unable to authenticate with provided credentials')
			raise serializers.ValidationError(msg, code='authenticate')

		attrs['user'] = user
		return attrs
```
### JSON Web Token Authentication
JSON Web Token is a fairly new standard which can be used for token-based authentication. Unlike the built-in TokenAuthentication scheme, JWT Authentication doesn't need to use a database to validate a token. A package for JWT authentication is djangorestframework-simplejwt which provides some features as well as a pluggable token blacklist app.
