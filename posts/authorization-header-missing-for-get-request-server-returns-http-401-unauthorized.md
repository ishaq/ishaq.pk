<!--
.. title: GET ریکوئسٹ میں Authorization ہیڈر نہیں ملتا، اور سرور غلطی نمبر 401 بھیجتا ہے
.. slug: authorization-header-missing-for-get-request-server-returns-http-401-unauthorized
.. date: 2014/05/06 17:49:54
.. tags: پروگرامنگ
.. link:
.. description:
.. type: text
-->

مجھے اس چیز نے ایک دو دفعہ خاصا پریشان کیا ہے، اور گوگل سرچ سے معلوم ہوتا ہے کہ میں اکیلا نہیں۔ دراصل میں ایک یو آر ایل اس طرح ٹائپ کر رہا تھا:

```
http://example.com/profile

```

نوٹ کریں کہ اس کے آخر میں کوئی سلیش ‪(/)‬نہیں ہے۔ درحقیقت مجھے:


```
http://example.com/profile/
```

ٹائپ کرنا چاہئے تھا، دیکھیں کہ اس کے آخر میں سلیش ‪(/)‬ہے۔


جب بھی میں غلط یو آر ایل کو کھولنے کی کوشش کرتا تھا، سرور مضمر طور پر مجھے صحیح یو آر ایل پر بھیج دیتا تھا۔ لیکن ایسا کرتے ہوئے Authorization Header راستے میں ہی کھو جاتا تھا۔ اور صحیح یو آر ایل سے `HTTP 401 Unauthorized`موصول ہوتا تھا۔

بظاہر بڑی سیدھی سی چیز ہے، لیکن کبھی سیدھی چیزیں ہی نظر آنے سے رہ جاتی ہیں۔

## اگر یہ مسئلہ آپ کو پروڈکشن میں آ رہا ہے (اور آپ نے Django REST Framework استعمال کیا ہے)##

اگر آپ کے پروجیکٹ میں Django REST Framework استعمال ہوتا ہے اور اس کو Apache mod‪_‬wsgi کے زریعے ڈپلائے کیا گیا ہے تو زہن میں رکھیے کہ ڈیفالٹ طور پر Django کو Authorization Header نہیں بھیجا جائے گا۔ اس فریم ورک کی [دستاویز کاری](http://www.django-rest-framework.org/api-guide/authentication#apache-mod_wsgi-specific-configuration) کہتی ہے:

> Note that if deploying to Apache using mod_wsgi, the authorization header is not passed through to a WSGI application by default, as it is assumed that authentication will be handled by Apache, rather than at an application level.

> If you are deploying to Apache, and using any non-session based authentication, you will need to explicitly configure mod_wsgi to pass the required headers through to the application. This can be done by specifying the WSGIPassAuthorization directive in the appropriate context and setting it to 'On'.

```apache
# this can go in either server config, virtual host, directory or .htaccess
WSGIPassAuthorization On
```

یعنی آپ کو Apache کی کنفگریشن فائل میں مناسب جگہ پر `WSGIPassAuthorization On`بتانا پڑے گا

**پس نوشت‪**‬

نیٹ ورک کوڈ کی ٹسٹنگ کے لیے اچھا سافٹ وئیر ہے۔ [Cocoa Rest Client](https://github.com/mmattozzi/cocoa-rest-client) 
