---
layout: post
title: "git log and datetime python"
date: 2024-01-18 15:13:04
categories: python
---

If you ever need to convert the git log output of something like:
```bash
git log --format='%cD'
```

To a `datetime` object in python here you go:

```python
date_format = '%a, %d %b %Y %H:%M:%S %z'
```

If you want to run through all of the commits and turn into a list, (for graphing maybe?)
here you go:

```python
def graph_activity_all_time(repository):
    dates = []
    date_objects = []

    # get all the commits dates:
    for j in os.popen(f"git log --format='%cD'").readlines():
        j = j.strip()
        dates.append(j)

    # Wed, 17 May 2023 11:37:51 -0500
    date_format = '%a, %d %b %Y %H:%M:%S %z'
    for k in dates:
        date_time_object = datetime.strptime(k, date_format)
        date_objects.append(date_time_object)
```

I spent way too long trying to figure this out, and hopefully this'll help someone in the future.
