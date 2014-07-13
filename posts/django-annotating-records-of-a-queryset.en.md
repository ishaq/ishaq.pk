<!--
.. title: Django: Annotating Records of a Queryset
.. slug: django-annotating-records-of-a-queryset
.. date: 05/30/14 18:25:06 UTC+05:00
.. tags: Django, Python, Programming
.. link:
.. description:
.. type: text
-->

Given a model structure like this (stripped down for clarity):

```python
from django.db import models

class Tag(models.Model):
    name = models.CharField(max_length=30, unique=True)
    is_deleted = models.BooleanField(default=False)

class Story(models.Model):
    text = models.TextField()
    is_deleted = models.BooleanField(default=False)

class StoryTag(models.Model):
    story = models.ForeignKey(Story)
    tag = models.ForeignKey(Tag)
    is_deleted = models.BooleanField(default=False)

```

**Requirement:** Display a list of most popular tags (along with corresponding stories count). We should only display non-deleted tags (`is_deleted` should be `False`). Also, only non-deleted stories should be counted.

One way to achieve it is:

```python
# 1. filter
tags_queryset = Tag.objects.filter(is_deleted = False, storytag__story__is_deleted = False)
tags_queryset = tags_queryset.distinct() # This is a bad idea, but we'll live with it, it's a small site anyway
# 2. annotate
tags_queryset = tags_queryset.annotate(num_stories=Count('storytag__story'))
# 3. order
tags_queryset = tags_queryset.order_by('-num_stories')

# 4. Test
for tag in tags_queryset:
    print "Tag: {0} - {1}".format(tag.name, tag.num_stories)

# output would be something like ('humor', 'scifi', 'kids' and 'suspence' are tags)
# humor - 24
# scifi - 43
# kids - 11
# suspence - 19

```

Documentation: [Django Docs on Aggregation](https://docs.djangoproject.com/en/dev/topics/db/aggregation/).
