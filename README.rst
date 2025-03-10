=====================================
Django SAML2 Authentication Made Easy
=====================================

:Author: Fang Li
:Version: Use 1.1.4 for Django <=1.9 and 2.x.x for Django >= 1.8

.. image:: https://img.shields.io/pypi/pyversions/django-saml2-auth.svg
    :target: https://pypi.python.org/pypi/django-saml2-auth

.. image:: https://img.shields.io/pypi/v/django-saml2-auth.svg
    :target: https://pypi.python.org/pypi/django-saml2-auth

.. image:: https://img.shields.io/pypi/dm/django-saml2-auth.svg
        :target: https://pypi.python.org/pypi/django-saml2-auth

This project aims to provide a dead simple way to integrate SAML2
Authentication into your Django powered app. Try it now, and get rid of the
complicated configuration of SAML.

Any SAML2 based SSO(Single-Sign-On) identity provider with dynamic metadata
configuration is supported by this Django plugin, for example Okta.



Donate
======

We accept your donations by clicking the awesome |star| instead of any physical transfer.

.. |star| image:: https://img.shields.io/github/stars/fangli/django-saml2-auth.svg?style=social&label=Star&maxAge=86400



Dependencies
============

This plugin is compatible with Django 1.6/1.7/1.8/1.9/1.10. The `pysaml2` Python
module is required.



Install
=======

You can install this plugin via `pip`:

.. code-block:: bash

    # pip install django_saml2_auth

or from source:

.. code-block:: bash

    # git clone https://github.com/fangli/django-saml2-auth
    # cd django-saml2-auth
    # python setup.py install

xmlsec is also required by pysaml2:

.. code-block:: bash

    # yum install xmlsec1
    // or
    # apt-get install xmlsec1


What does this plugin do?
=========================

This plugin takes over Django's login page and redirect the user to a SAML2
SSO authentication service. Once the user is logged in and redirected back,
the plugin will check if the user is already in the system. If not, the user
will be created using Django's default UserModel, otherwise the user will be
redirected to their last visited page.



How to use?
===========

#. Import the views module in your root urls.py

    .. code-block:: python

        import django_saml2_auth.views

#. Override the default login page in the root urls.py file, by adding these
   lines **BEFORE** any `urlpatterns`:

    .. code-block:: python

        # These are the SAML2 related URLs. You can change "^saml2_auth/" regex to
        # any path you want, like "^sso_auth/", "^sso_login/", etc. (required)
        url(r'^saml2_auth/', include('django_saml2_auth.urls')),

        # The following line will replace the default user login with SAML2 (optional)
        # If you want to specific the after-login-redirect-URL, use parameter "?next=/the/path/you/want"
        # with this view.
        url(r'^accounts/login/$', django_saml2_auth.views.signin),

        # The following line will replace the admin login with SAML2 (optional)
        # If you want to specific the after-login-redirect-URL, use parameter "?next=/the/path/you/want"
        # with this view.
        url(r'^admin/login/$', django_saml2_auth.views.signin),

#. Add 'django_saml2_auth' to INSTALLED_APPS

    .. code-block:: python

        INSTALLED_APPS = [
            '...',
            'django_saml2_auth',
        ]

#. In settings.py, add the SAML2 related configuration.

    Please note, the only required setting is **METADATA_AUTO_CONF_URL**.
    The following block shows all required and optional configuration settings
    and their default values.

    .. code-block:: python

        SAML2_AUTH = {
            # Required setting
            'METADATA_AUTO_CONF_URL': '[The auto(dynamic) metadata configuration URL of SAML2]',

            # Optional settings below
            'DEFAULT_NEXT_URL': '/admin',  # Custom target redirect URL after the user get logged in. Default to /admin if not set. This setting will be overwritten if you have parameter ?next= specificed in the login URL.
            'NEW_USER_PROFILE': {
                'USER_GROUPS': [],  # The default group name when a new user logs in
                'ACTIVE_STATUS': True,  # The default active status for new users
                'STAFF_STATUS': True,  # The staff status for new users
                'SUPERUSER_STATUS': False,  # The superuser status for new users
            },
            'ATTRIBUTES_MAP': {  # Change Email/UserName/FirstName/LastName to corresponding SAML2 userprofile attributes.
                'email': 'Email',
                'username': 'UserName',
                'first_name': 'FirstName',
                'last_name': 'LastName',
            },
            'TRIGGER': {
                'CREATE_USER': 'path.to.your.new.user.hook.method',
                'BEFORE_LOGIN': 'path.to.your.login.hook.method',
                'AFTER_LOGIN': 'path.to.your.login.hook.method',
            },
            'SP_PROPERTIES': {
                'allow_unsolicited': True,
                'authn_requests_signed': False,
                'logout_requests_signed': True,
                'want_assertions_signed': True,
                'want_response_signed': False,
            },
            'SERVICE_PROPERTIES': {
                'cert_file': 'cert.pem',
                'contact_person': '"contact_person": [{
                                    "givenname": "Roland",
                                    "surname": "Hedberg",
                                    "phone": "+46 90510",
                                    "mail": "roland@example.com",
                                    "type": "technical",
                                    }]',
                'debug': 1,
                'entityid': 'http://www.hostname.com',
                'key_file': 'key.pem',
                'organization': '"organization": {
                                 "display_name":["Example identities"]
                                }',
                'xmlsec_binary': '/usr/local/bin/xmlsec1',
                'valid_for': 24,
            },
        }

#. In your SAML2 SSO identity provider, set the Single-sign-on URL and Audience
   URI(SP Entity ID) to http://your-domain/saml2_auth/acs/


Explanation
-----------

**METADATA_AUTO_CONF_URL** Auto SAML2 metadata configuration URL

**NEW_USER_PROFILE** Default settings for newly created users

**ATTRIBUTES_MAP** Mapping of Django user attributes to SAML2 user attributes

**TRIGGER** Hooks to trigger additional actions during user login and creation
flows. These TRIGGER hooks are strings containing a `dotted module name <https://docs.python.org/3/tutorial/modules.html#packages>`_
which point to a method to be called. The referenced method should accept a
single argument which is a dictionary of attributes and values sent by the
identity provider, representing the user's identity.

**TRIGGER.CREATE_USER** A method to be called upon new user creation. This
method will be called before the new user is logged in and after the user's
record is created. This method should accept ONE parameter of user dict.

**TRIGGER.BEFORE_LOGIN** A method to be called when an existing user logs in.
This method will be called before the user is logged in and after user
attributes are returned by the SAML2 identity provider. This method should accept ONE parameter of user dict.

**TRIGGER.AFTER_LOGIN** A method to be called when an existing user logs in.
This method will be called after the user is logged in and after user
attributes are returned by the SAML2 identity provider. This method should accept ONE parameter of user dict.


Customize
=========

The default permission `denied` page and user `welcome` page can be
overridden.

To override these pages put a template named 'django_saml2_auth/welcome.html'
or 'django_saml2_auth/denied.html' in your project's template folder.

If a 'django_saml2_auth/welcome.html' template exists, that page will be shown
to the user upon login instead of the user being redirected to the previous
visited page. This welcome page can contain some first-visit notes and welcome
words. The `Django user object <https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.User>`_
is available within the template as the `user` template variable.

To enable a logout page, add the following lines to urls.py, before any
`urlpatterns`:

.. code-block:: python

    # The following line will replace the default user logout with the signout page (optional)
    url(r'^accounts/logout/$', 'django_saml2_auth.views.signout'),

    # The following line will replace the default admin user logout with the signout page (optional)
    url(r'^admin/logout/$', 'django_saml2_auth.views.signout'),

To override the built in signout page put a template named
'django_saml2_auth/signout.html' in your project's template folder.

If your SAML2 identity provider uses user attribute names other than the
defaults listed in the `settings.py` `ATTRIBUTES_MAP`, update them in
`settings.py`.


For Okta Users
==============

I created this plugin originally for Okta.

The METADATA_AUTO_CONF_URL needed in `settings.py` can be found in the Okta
web UI by navigating to the SAML2 app's `Sign On` tab, in the Settings box.
You should see :

`Identity Provider metadata is available if this application supports dynamic configuration.`

The `Identity Provider metadata` link is the METADATA_AUTO_CONF_URL.


How to Contribute
=================

#. Check for open issues or open a fresh issue to start a discussion around a feature idea or a bug.
#. Fork `the repository`_ on GitHub to start making your changes to the **master** branch (or branch off of it).
#. Write a test which shows that the bug was fixed or that the feature works as expected.
#. Send a pull request and bug the maintainer until it gets merged and published. :) Make sure to add yourself to AUTHORS_.

.. _`the repository`: http://github.com/fangli/django-saml2-auth
.. _AUTHORS: https://github.com/fangli/django-saml2-auth/blob/master/AUTHORS.rst


Release Log
===========

2.1.0: Add DEFAULT_NEXT_URL. Issue #19.

2.0.4: Fixed compatibility with Windows.

2.0.3: Fixed a vulnerabilities in the login flow, thanks qwrrty.

2.0.1: Add support for Django 1.10

1.1.4: Fixed urllib bug

1.1.2: Added support for Python 2.7/3.x

1.1.0: Added support for Django 1.6/1.7/1.8/1.9

1.0.4: Fixed English grammar mistakes
