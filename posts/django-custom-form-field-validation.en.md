<!--
.. title: Django: Custom Form Field Validation
.. slug: django-custom-form-field-validation
.. date: 05/30/14 18:47:00 UTC+05:00
.. tags: Django, Python, Programming
.. link:
.. description:
.. type: text
-->

Given a model structure like this (stripped down for clarity):

```python
from django.db import models
from django.conf import settings

class TouristProfile(models.Model):
    owner = models.OneToOneField(settings.AUTH_USER_MODEL,
        related_name='pilgrim_profile')
    tour_start_date = models.DateField(null=True, blank=True)
    tour_end_date = models.DateField(null=True, blank=True)
```

**Requirement:** Say we have a *View Tourist Profile* page. We want to let a site visitor select a tourist and view his profile. *But* we can only display a tourist profile for users who have a created a `TouristProfile` *and* have specified a `tour_start_date`. So, all tourists who either did not create a `TouristProfile` or did not specify a `tour_start_date` are not valid choices.

One way to do it is through custom field validation, as shown below:

```python
from django.contrib.auth import get_user_model
from django.utils.translation import ugettext as _
from myapp.models import TouristProfile

class TouristSearchForm(forms.Form):
    tourists_queryset = get_user_model().objects.filter(is_active=True).order_by('username')
    tourist = forms.ModelChoiceField(queryset=tourists_queryset)

    def clean_tourist(self):
        data = self.cleaned_data['tourist']
        try:
            profile = TouristProfile.objects.get(owner=self.cleaned_data['tourist'])
        except TouristProfile.DoesNotExist:
            raise forms.ValidationError(_("This tourist has not created a profile"),
                code="profile_null")
        if profile.tour_start_date is None:
            raise forms.ValidationError(_("This tourist has not specified tour start date"),
                code="tour_start_date_null")
        return data # data should be returned whether we modified it or not

```

When rendered in a template, this form would display a list of all users in the database.

In the view code, if you call `is_valid()` on this form and:

* if user selected a tourist who is missing `TouristProfile`, it will raise the error: `This tourist has not created a profile`.
* if user selected a tourist who created a profile but did not specify `tour_start_date`, it will raise the error: `This tourist has not specified tour start date`.

**NOTE:** we could have just filtered `tourist` (in `TouristSearchForm`) to contain valid choices only. But my use case required me to list all the users.

Documentation Link: [Django Docs on Form and Field  Validation](https://docs.djangoproject.com/en/dev/ref/forms/validation/)
