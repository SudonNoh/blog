# 목표
1. AWS DB와 연결해서 배포
2. social login 기능 추가

## First Day

### User
### Creating the User model
1. Secret Key와 settings.py 파일을 분리하기 위해 <b>*blog/secrests.json*</b>을 manage.py이 있는 project 상위 폴더에 만듭니다. Secret Key가 git에 upload할 때 포함되지 않도록 <b>*blog/.gitignore*</b> 파일에 추가해줍니다.

2. User, 인증 관련 app을 만들기 위해 authentiction app을 만들어줍니다.  
`django-admin startapp authentication`

3. BaseUserManager를 만들기 위해 <b>*blog/authentication/managers.py*</b> 파일을 authentication app 안에 만들어줍니다. 그 안에 class Usermanager를 정의합니다.

4. User 모델을 만들기 위해 <b>*blog/authentication/models.py*</b> 파일을 열어 수정해줍니다. 이 과정에서 <b>*blog/setting/settings.py*</b> 파일을 열어 AUTH_USER_MODEL을 설정해줍니다. 이후 제대로 작동하는지 super user를 생성해 확인합니다.  
`python manage.py createsuperuser`

----
>여기에서 django.db.utils.OperationalError: no such table: 오류가 발생했는데요. createsuperuser를 실행하는 과정에서 db의 table을 지웠고, db.sqlite3 파일을 지워버리면서 생긴 오류 같았습니다.<br><br>이를 해결하기 위해 db.sqlite3 파일을 삭제하고, makemigrations을 우선 진행했습니다. 다음으로 <br>`python manage.py migrate --run-syncdb` 를 실행했습니다. `--run-syncdb` 옵션은 migration을 하지 않고 앱에 대한 테이블을 만드는 것입니다.
----
>django shell에서 model들을 한번에 import 하고 사용하기 위해서 extensions를 설치해줄 필요가 있습니다.<br> `pip install django-extensions` 으로 설치하고 터미널에<br>`python manage.py shell_plus`를 입력해주면 django와 관련된 모듈들이 import 되는 것을 확인할 수 있습니다.<br><br>이제 본인이 정의한 모델명으로 객체에 접근할 수 있습니다. 다음 명령어로 조회해보시기 바랍니다. `User.objects.all()` 여기서 User는 각자 정의한 모델명이므로 작성자에 따라 다를 수 있습니다.
----

## Second Day

5. superuser를 만들어 테스트가 끝나면 인증 부분을 수정해보도록 하겠습니다. 먼저 <b>*blog/authentication/models.py*</b> 파일에 `@property`를 추가해 token을 만들기 위한 함수를 정의해줍니다. 제대로 적용이 되었는지 확인해봅니다. 위에서 언급한 `shell_plus`로 shell을 열고, `User.objects.first().token`으로 token이 잘 나오는지 확인합니다.
----
>pyjwt의 버전이 1.7.1 이하이면 decode를 적용시켜야 합니다. return 부분에 token.decode('utf-8')을 작성해줍니다. 
----
### Registering new users
6. 이제 모델의 내용을 create하거나 update할 때 내용을 직렬화 하기 위한 serializer를 만들어보도록 하겠습니다. 우선 폴더를 하나 더 만들도록 하겠습니다. <b>*blog/authentication/api*</b> api 폴더를 만들어서 serializer 혹은 view, 각 앱에서 사용되는 modelue들을 모아두도록 하겠습니다. 다음으로 해당 폴더에 serializer를 만들도록 하겠습니다. <b>*blog/authentication/api/serializers.py*</b>

7. 실질적으로 새로운 user를 create할 때 사용되는 View를 작성해보도록 하겠습니다. 마찬가지로 <b>*blog/authentication/api/views.py*</b> 파일을 만들어주겠습니다.

8. 지금까지 만든 api들이 정상적으로 작동하는지 확인하기 위해 url 파일에서 바꾸어야 하는 부분을 바꾸도록 하겠습니다. 우선 <b>*blog/authentication/api/urls.py*</b> 파일을 만들고 수정해보도록 하겠습니다. 다음으로는 <b>*blog/setting/urls.py*</b> 파일을 수정하도록 하겠습니다.<br><br>그 후에 postman을 이용해서 사용자 등록이 잘 작동하는지 확인해보겠습니다.

9. 다음은 Rendering 과정을 처리하기 위해 <b>*blog/authentication/api/renderers.py*</b> 파일을 만듭니다. 만든 renderer를 <b>*blog/authentication/api/views.py*</b> 파일에 import 합니다.

### Logging users in
10. 등록된 ID로 Login하는 기능을 만들기 위해 serializer를 먼저 만들어줍니다. 우선 <b>*blog/authentication/api/serializers.py*</b> 파일을 열어줍니다. 파일 내부에 있는 `RegistrationSerializer` 아래에 `LoginSerializer`를 만들어줍니다.

11. 이번엔 View를 수정해보도록 하겠습니다. <b>*blog/authentication/api/views.py*</b> 파일을 열어 `LoginAPIView`를 추가해줍니다. 추가한 후 <b>*blog/authentication/api/urls.py*</b> 파일을 열어 url을 설정합니다. 이후 postman에서 login이 정상적으로 이루어지는지 확인해봅니다. <br><br>이 과정에서 post가 정상적으로 이루어지지 않는 경우 "non_field_errors" 라는 오류를 반환합니다. 여기에는 한 가지 문제가 있는데요.<br><br> 일반적으로 이 오류는 serializer가 유효성 검사를 실패하게 만든 모든 필드에 해당됩니다. 즉, 포괄적인 전체 error를 보여줄 때 설정됩니다. 우리가 만든 validator의 경우 validate_email과 같은 필드별 method 대신에 validate method 자체를 override했기 때문에 DRF는 오류에서 반환할 필드를 알지 못하기 때문에 특정 field error를 반환하지 못하고, "non_field_errors"를 반환한 것 입니다.<br><br> client는 보여지는 error(여기서는 "non_field_errors")를 사용해 표시하기 때문에 저는 간단하게 "non_field_errors"를 "error"로 변경하도록 하겠습니다. <br><br> 이 문제를 해결하기 위해 기본 error 처리를 override하도록 하겠습니다.

## Third day
### Overriding EXCEPTION_HANDLER and NON_FIELD_ERRORS_KEY

12. DRF setting 중 하나가 EXCEPTION_HANDLER입니다. 기본 exception handler는 단순하게 오류 dictionary를 반환합니다. 저는 EXCEPTION_HANDLER를 override 하고, NON_FIELD_ERRORS_KEY를 앞서 언급한대로 override하도록 하겠습니다.<br><br> 우선 <b>*blog/core*</b> 라는 폴더를 만들고 그 안에 <b>*blog/core/exceptions.py*</b> 파일을 만들어 줍니다.

13. 그 다음으로 <b>*blog/setting/settings.py*</b> 파일에서 새로운 setting을 추가해줍니다. 그 다음 postman에서 email/password를 invalid data로 전송해봅니다. 그러면 다음과 같은 error message가 뜹니다.
```json
{
    "user": {
        "errors": {
            "error": [
                "A user with this email and password was not found."
            ]
        }
    }
}
```

14. 현재 user key 아래에 모든 errors key, error가 존재합니다. 이런 형태는 좋지 못하기 때문에 수정해줄 필요가 있습니다. Rendering할 때 error가 발생하면 user key에서 보여지는 것이 아닌 errors만 보여지도록 하겠습니다. <b>*blog/authentication/api/renderers.py*</b> 파일을 열어서 일부분 추가해줍니다. 그 다음 다시 login 과정에서 invalid data를 보내면 아래와 같은 결과를 얻을 수 있습니다.
```json
{
    "errors": {
        "error": [
            "A user with this email and password was not found."
        ]
    }
}
```

### Retrieving and updating users

15. 이번에는 사용자들의 정보를 업데이트하고, 사용자의 정보를 검색하는 기능을 만들어보도록 하겠습니다. 우선 <b>*blog/authetication/api/serializers.py*</b> 파일을 열어줍니다. 그 다음 update 부분을 추가해주도록 하겠습니다.

16. 다음 단계로 View를 만들어보도록 하겠습니다. <b>*blog/authentication/api/views.py*</b> 파일을 열고, 그 안에 RetrieveUpdateAPIView를 만들어넣도록 하겠습니다. 그 다음 <b>*blog/authentication/api/urls.py*</b> 파일을 열어 get, patch 부분을 사용할 수 있도록 url을 작성해주도록 하겠습니다.<br><br>모든 과정을 끝낸 뒤에 postman에서 get을 시도해보겠습니다. 그 결과 아래와 같은 값을 받았습니다.
```json
{
    "user": {
        "detail": "Authentication credentials were not provided."
    }
}
```

17. 위와 같은 결과값을 받은 이유는 Django나 DRF가 기보적으로 JWT인증을 지원하지 않기 때문입니다. 이를 해결하기 위해 custom backend를 만들어야 합니다. <b>*blog/authentication/backends.py*</b> 파일을 만들고 그 안에 내용을 채워주도록 하겠습니다.

18. 그 다음 <b>*blog/stting/settings.py*</b> 파일을 열어 `REST_FRAMEWORK` 부분에 `DEFAULT_AUTHENTICATION_CLASSES`를 추가해주도록 하겠습니다. 

### Profile
### Creating the Profile model

19. 이번 단계에서는 Profile model을 만들고자 합니다. User model이 있음에도 불구하고 Profile model을 만드는 이유에 대해서 얘기해보려고 합니다. 우선 User model은 인증(authentication) 및 권한(permissions) 부여를 위한 것입니다. User model의 역할은 사용자가 접근하려는 항목에 접근할 수 있도록 하는 것입니다. 이와 대조적으로 Profile model은 사용자의 정보를 UI에 보여주는 역할을 합니다. 따라서 공개해도 되는 정보들을 Profile model에 정리해서 사용하려고 합니다.<br><br>이 과정을 위해 첫번째로 profile app을 시작해야합니다.<br>터미널에 `python manage.py startapp profiles`를 실행해줍니다. <b>*blog/profiles/models.py*</b> 파일을 열고, Profile model을 작성하도록 하겠습니다.

20. Profile model을 만들면서 한 가지 의문점이 생겼습니다. User model과 마찬가지로 `created_at`, `updated_at` field를 만들었다는 것입니다. 중복해서 두번의 코드를 작성한 것인데요. 앞으로 다른 모델을 만들어도 이 두 가지 field(`created_at`, `updated_at`)들은 계속 추가될 것입니다. <br><br>우리는 코드를 중복해서 작성하는 것을 방지하기 위해 상속을 활용하도록 하겠습니다. 위 두 가지 field를 따로 model로 만들어 관리하고, User Model과 Profile model에서 상속받아 사용하도록 합니다.<br><b>*blog/core/models.py*</b> 파일을 만들어 앞으로 중복해서 사용될 model을 미리 만들어두도록 하겠습니다.

21. 그 다음 <b>*blog/profiles/models.py*</b> 파일과 <b>*blog/authentication/models.py*</b> 파일을 열어 각각 Profile model과 User model을 수정해주도록 하겠습니다.

22. Profile model에서 우리는 User model과 Profile model 사이에 one-to-one 관계를 만들었습니다. 이것이 완벽하게 호환이 되어 User가 만들어지면 Profile이 자동으로 만들어지면 정말 좋겠지만 안타깝게도 Django에서는 User가 만들어질 때 Profile도 따로 만들어야 합니다.<br><br>우리는 이를 자동화시키기 위해 `Django's Signals framework`를 사용하도록 하겠습니다. 특히 post_save signal을 사용할 예정인데 우리는 이것을 활용해 User instance가 만들어지면 Profile instance도 자동으로 만들어지도록 만들겠습니다.<br><br>이를 위해 <b>*blog/authentication/signals.py*</b> 파일을 만들어 줍니다.

23. signal 코드를 모두 작성했으면 profile object들을 만들어낼겁니다. 하지만 Django는 이 signal을 default로 실행하지는 않습니다. 따라서 default로 실행하도록 만들 필요가 있습니다. <b>*blog/authentication/apps.py*</b> 파일로 가서 코드를 추가해줍니다. <br><br>코드 추가가 완료되면 User object를 만들 때 Profile object가 자동으로 만들어지는 것을 확인할 수 있을 겁니다. 이 부분을 `shell`을 통해 확인해보도록 하겠습니다. 우선 <b>*blog/setting/settings.py*</b> 파일을 열고 `INSTALLED_APPS` 부분에 profiles app을 등록해주도록 하겠습니다. 다음으로 아래 과정을 따라서 확인해주시면 됩니다.<br><br>`python manage.py makemigrations`<br><br>`python manage.py migrate`<br><br>`python manage.py shell_plus` <br>여기서 shell_plus가 실행되지 않으시는 분들은 위에 언급된 django-extensions를 설치하고 오시기 바랍니다.<br><br>`>>> user = User.objects.first()`<br>`>>>user.profile`<br>`<Prfile: username>`

24. 이제는 serializer를 만들도록 하겠습니다. <b>*blog/profiles/serializers.py*</b> 파일을 만들고 코드를 작성합니다. 

### Rendering Profile objects

25. `ProfileJSONRenderer`를 만들 차례입니다. 앞서 만들었던 `UserJSONRenderer`와 유사합니다. 그렇기 때문에 TimestampedModel을 만들었던 것처럼 이번에는 Renderer를 만들어 상속해보도록 하겠습니다. <b>*blog/core/renderers.py*</b> 파일을 만들어줍니다. 그리고 코드를 입력해줍니다.<br><br>`UserJSONRenderer`와 `BaseJSONRenderer`는 차이가 있습니다. 그 차이에 대해 설명해보도록 하겠습니다. <br><br>첫번째로 `UserJSONRenderer`에서는 `object_label`이라는 속성을 따로 명시하지는 않았습니다. 그 이유는 `UserJSONRenderer`에 대한 object label이 무엇인지 이미 알고 있었기 때문입니다.<br><br>하지만 이 경우에는 object label이 `BaseJSONRenderer`를 상속받는 대상에 따라 달라지기 때문입니다. 더 유용하게 만들기 위해서 우리는 `object_label`을 동적으로 설정되도록 허용하고 기본적으로 object 값을 사용하도록 했습니다.<br><br>두 번째로 `UserJSONRenderer`는 JWT decoding(`data['token'] = token.decode('utf-8')`)에 대해 생각해야 했습니다. 이것은 다른 Renderer에는 없는 UserJSONRenderer에만 해당되는 요구사항입니다. 이것을 사용하기 위해 `BaseJSONRenderer`에 포함해서 작성하는 것은 무의미합니다. 위 사항들을 해결하기 위해 `UserJSONRenderer`를 수정하도록 하겠습니다.

26. 우선 간단하게 profiles app의 renderer를 만들기 위해 <b>*blog/profiles/api/renderers.py*</b> 파일을 열어 코드를 입력합니다. 다음으로 위 사항을 수정하기 위해 <b>*blog/authetication/api/renderers.py*</b> 파일을 열어줍니다. `BaseJSONRenderer`부분에서 처리하는 부분은 삭제하고 `UserJSONRenderer`에서 처리해야 하는 부분을 남겨줍니다. 이제 잘 작동이 되는지 postman에서 Current User를 확인해서 알아보도록 하겠습니다. 성공적으로 값을 받아옴을 확인했습니다.
```json
{
    "user": {
        "email": "client@blog.com",
        "username": "tony",
        "birth_date": "1992-01-01",
        "phone_number": "010-9912-0220",
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MiwiZXhwIjoxNjQzMDE5MzAyfQ.mBP0-aVpEz8KiU7kIYdBIVetlfAFcZa-HFwFoB_pMck"
    }
}
```

## Forth day

### ProfileRetrieveAPIView

27. 이번 파트에서는 `ProfileRetriveAPIView`를 만들어보도록 하겠습니다. <b>*blog/profiles/api/views.py*</b> 파일을 만들고 코드를 입력합니다. 이번 코드에서 request 받은 profile이 존재하지 않을 경우, `Profile.DoesNotExist` 이 나타나도록 했습니다. 하지만 client가 `Profile.DoesNotExist`를 응답으로 받았을 때, 확인할 수 있는 메세지가 명확하게 나와있지 않습니다. 이것을 해결하기 위해서 <b>*blog/profiles/api/exceptions.py*</b> 파일을 만들어 코드를 작성해보겠습니다.<br><br>`exceptions.py` 파일 작성이 끝나면 이제 이것을 <b>*blog/core/exceptions.py*</b> 파일의 `core_exception_handler` 부분에 추가해주어야 합니다. 또 이렇게 추가된 `ProfileDoesNotExist` exception을 <b>*blog/profiles/api/views.py*</b>에 추가해주어야 합니다. 

28. 이제 해당 `ProfileRetriveAPIView`를 url에 입력해주어야 합니다. <b>*blog/profiles/api/urls.py*</b> 파일을 만들고 코드를 입력합니다. 그 다음으로 <b>*blog/setting/urls.py*</b> 파일을 열고 url을 연결하는 코드를 입력합니다. 이번 url에서는 username을 parameter로 보냅니다. 이것을 받아 view에서 검색 후 serializer를 거쳐 데이터로 반환합니다.

### Updating UserRetrieveUpdateAPIView

29. 이제 User model과 Profile model의 객체(object) 생성(create/post) 부분은 끝났습니다. 이번에는 User model의 객체를 Update 할 때, Profile model의 요소도 함께 update 할 수 있도록 `UserRetrieveUpdateAPIView`를 수정하겠습니다. <b>*blog/authentication/api/views.py*</b> 파일을 열고 코드를 수정해줍니다. 코드 작성이 완료되면 비로소 email, password, introduce 등의 update에 대한 동일한 endpoint를 갖게 됩니다. <br><br>이어서 `UserSerializer`도 수정하겠습니다. <b>*blog/authentication/api/serializers.py*</b> 파일을 열고 코드를 수정해줍니다.

### Articles

지금까지 User model, Profile model에 대한 post, get, update 기능을 만들어보았습니다. 이제 client들은 자신들의 id를 만들고, log in 하고, 다른 사용자들의 profile도 볼 수 있는 상태가 되었습니다. 이번 파트에서부터는 posting system을 만들어 보도록 하겠습니다. 인증받은 사용자들은 글을 쓰고 다른 유저들은 그 글을 읽을 수 있습니다.

### Creating the Article model

30. 먼저 앞에서부터 꾸준히 해왔던 것처럼 model을 먼저 구성해보도록 하겠습니다. <b>*blog/articles/models.py*</b> 파일을 만들고 코드를 입력합니다. 이제 <b>*blog/setting/settings.py*</b> 파일에 articles app을 추가해준 뒤, migration을 진행해줍니다.<br><br>`python manage.py makemigrations`<br>`python manage.py migrate`

31. Serializer를 만들도록 하겠습니다. <b>*blog/articles/api/serializers.py*</b> 파일을 만들고 코드를 입력합니다. 다음으로 <b>*blog/articles/api/renderers.py*</b> 파일을 추가로 만들어줍니다.

32. 이어서 View를 만들도록 하겠습니다. <b>*blog/articles/api/views.py*</b> 파일을 만들어 view 코드를 작성하겠습니다. <br><br>GenericViewSet 클래스는 GenericAPIView를 상속하고, get_object, get_queryset meothod들을 기본 set으로 제공하고 다른 generic view의 기본 행동을 제공합니다. 하지만 어떤 action도 default 값으로 제공되지 않고 있습니다.<br><br>따라서 GenericViewSet 클래스를 사용하기 위해 사용자는 해당 클래스를 override하고 request 받은 mixin 클래스를 override하거나 명식적으로 action을 구현해야 합니다.

33. url을 연결시키기 위해 <b>*blog/articles/api/urls.py*</b> 파일을 만들어 코드를 입력하겠습니다. 여기에서 router를 사용할 예정입니다. 다음으로 <b>*blog/setting/urls.py*</b> 파일을 열어 articles app과 연결하는 코드를 입력해줍니다.


34. 