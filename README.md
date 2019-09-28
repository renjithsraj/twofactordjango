

# twofactordjango

This Project will have the Implementation of TOTP validation with google authenticator for Django admin users. Main Aim of the project is add additional security feature for the admin users with TOTP validation.

clone the directory and installed below packages and run it.

[Demo Project]: https://github.com/renjithsraj/twofactordjango



#### How it's works:

------

The projects is works well with new version django 2.2.5 version and python 3.7, postgresql. please follow below steps to integrate two factor authentication in django based application. To achieve this  we are using  

***django-two-factor-auth*** this package.



#### pre qualities requirements

------

1. Django 2.X version
2. Python 3.X
3. postgresql     # for sqlite3 getting some error
4.  django-two-factor-auth



#### Configuration django-two-factor-auth with Django project

------



###### Basic config

```shell
(.venv1) hp@daemon:~/MYPROJECTS$ django-admin.py startproject demoproject

(.venv1) hp@daemon:~/MYPROJECTS$ pip install django-two-factor-auth
```



###### demoproject/demoproject/settings.py

```python


Make sure that the requirements already installed on your vitualenviornment

# Make sure the config register on top of the list in INSRALLED_APPS

INSTALLED_APPS = [
    'django_otp',
    'django_otp.plugins.otp_static',
    'django_otp.plugins.otp_totp',
    'two_factor',
    '............',
    # django apps register below
]

MIDDLEWARE = [
    '-------------------------------------------------------',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django_otp.middleware.OTPMiddleware',
    '--------------------------------------------------------'
]

TWO_FACTOR_FORCE_OTP_ADMIN = True
LOGIN_URL = 'two_factor:login'
LOGIN_REDIRECT_URL = '/admin'  # Redirect admin dashboard
```

Migrate all your installed apps

```
(.venv1) hp@daemon:~/MYPROJECTS$ python manage.py migrate
```

###### demoproject/demoproject/urls.py

```python
from django.contrib import admin
from django.urls import path, include, re_path
from two_factor.admin import AdminSiteOTPRequired
from two_factor.urls import urlpatterns as tf_urls

from django.conf import settings
from django.http import  HttpResponseRedirect
from django.contrib.auth import REDIRECT_FIELD_NAME
from django.contrib.auth.views import redirect_to_login
from django.shortcuts import resolve_url
from django.urls import reverse
from django.utils.http import is_safe_url
from two_factor.admin import AdminSiteOTPRequired, AdminSiteOTPRequiredMixin


class AdminSiteOTPRequiredMixinRedirSetup(AdminSiteOTPRequired):
    def login(self, request, extra_context=None):
        redirect_to = request.POST.get(
            REDIRECT_FIELD_NAME, request.GET.get(REDIRECT_FIELD_NAME)
        )
        # For users not yet verified the AdminSiteOTPRequired.has_permission
        # will fail. So use the standard admin has_permission check:
        # (is_active and is_staff) and then check for verification.
        # Go to index if they pass, otherwise make them setup OTP device.
        if request.method == "GET" and super(
            AdminSiteOTPRequiredMixin, self
        ).has_permission(request):
            # Already logged-in and verified by OTP
            if request.user.is_verified():
                # User has permission
                index_path = reverse("admin:index", current_app=self.name)
            else:
                # User has permission but no OTP set:
                index_path = reverse("two_factor:setup", current_app=self.name)
            return HttpResponseRedirect(index_path)

        if not redirect_to or not is_safe_url(
            url=redirect_to, allowed_hosts=[request.get_host()]
        ):
            redirect_to = resolve_url(settings.LOGIN_REDIRECT_URL)

        return redirect_to_login(redirect_to)
from django.contrib import admin
admin.site.__class__ = AdminSiteOTPRequiredMixinRedirSetup

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include(tf_urls, "two_factor")),

]
```



###### Template config : This changes I have made (you can't  find it anywhere)

templates/two_factor/

1. [_base.html](https://github.com/renjithsraj/twofactordjango/blob/master/templates/two_factor/_base.html)

2. [_base_focus.html](https://github.com/renjithsraj/twofactordjango/blob/master/templates/two_factor/_base_focus.html)



#### User configuration

Create super user account with following command

```
$ python manage.py createsuperuser
```

Once  you have created superuser, login with the admin dashboard, the  **django-two-factor-auth**  will force you to enable two factor authentication.



Screenshots for integrated app:

![Screen Shot 2019-09-27 at 5 57 38 PM](https://user-images.githubusercontent.com/8171465/65769462-389e2680-e151-11e9-8245-f14b09a60207.png)
![Screen Shot 2019-09-27 at 5 57 52 PM](https://user-images.githubusercontent.com/8171465/65769463-3936bd00-e151-11e9-8b25-7628c166d5b2.png)
![Screen Shot 2019-09-27 at 5 58 45 PM](https://user-images.githubusercontent.com/8171465/65769464-39cf5380-e151-11e9-9d92-68285e7e6f7d.png)
![Screen Shot 2019-09-27 at 5 58 55 PM](https://user-images.githubusercontent.com/8171465/65769465-3a67ea00-e151-11e9-8b20-b93520796b3e.png)
![Screen Shot 2019-09-27 at 5 59 16 PM](https://user-images.githubusercontent.com/8171465/65769466-3a67ea00-e151-11e9-8aca-832d5dd085f9.png)
![Screen Shot 2019-09-27 at 5 59 29 PM](https://user-images.githubusercontent.com/8171465/65769467-3a67ea00-e151-11e9-9f72-596ab8649d56.png)
![Screen Shot 2019-09-27 at 5 59 39 PM](https://user-images.githubusercontent.com/8171465/65769468-3a67ea00-e151-11e9-8d28-3d909fca804e.png)
[Screen Shot 2019-09-27 at 6 00 16 PM](https://user-images.githubusercontent.com/8171465/65769470-3b008080-e151-11e9-8bdf-8bf572ec636e.png)
