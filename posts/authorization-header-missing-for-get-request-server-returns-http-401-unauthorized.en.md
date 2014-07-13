<!--
.. title: Authorization Header Missing for GET Request, Server returns HTTP 401 Unauthorized
.. slug: authorization-header-missing-for-get-request-server-returns-http-401-unauthorized
.. date: 2014/05/06 17:49:54
.. tags: Programming
.. link:
.. description:
.. type: text
-->

This has troubled me a few times and quick Google search reveals there may be other people baffled by the same issue. I had mistakenly used a URL like:

```
http://example.com/profile  (no trailing slash, WRONG)

```

instead of

```
http://example.com/profile/ (has a trailing slash, CORRECT)
```

Whenever I hit the wrong URL, the server implicitly redirected the request to the correct. In the redirection process however, it dropped the authorization header and target URL returned `HTTP 401 Unauthorized`.

It seems straight forward in hindsight but can cause some headache.

## If you see this error in production (Django REST Framework)##

If project uses Django REST Framework and is deployed with Apache mod_wsgi, Authorization header will *not* be possed to django by default. From [DRF docs](http://www.django-rest-framework.org/api-guide/authentication#apache-mod_wsgi-specific-configuration)

> Note that if deploying to Apache using mod_wsgi, the authorization header is not passed through to a WSGI application by default, as it is assumed that authentication will be handled by Apache, rather than at an application level.

> If you are deploying to Apache, and using any non-session based authentication, you will need to explicitly configure mod_wsgi to pass the required headers through to the application. This can be done by specifying the WSGIPassAuthorization directive in the appropriate context and setting it to 'On'.

```apache
# this can go in either server config, virtual host, directory or .htaccess
WSGIPassAuthorization On
```

**PS:** [Cocoa Rest Client](https://github.com/mmattozzi/cocoa-rest-client) is a good utility to test HTTP requests.
