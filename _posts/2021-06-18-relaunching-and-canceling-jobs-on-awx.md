---
layout: post
title: "Relaunching and canceling jobs on AWX"
date: 2021-06-18 15:11:10
categories: awx sysadmin ansible python
---

As my team has been working more and more with [AWX][awx] we've had to do more
and more maintenance and administration on the system. One of the oddites with
job maintenance is that there is no "relaunch" button. If you look below you
can see what I mean:

IMAGE-OF-NO-RELAUNCH

We spent some time trying to figure out how to do this, and noticed quickly that
there was no CLI version. So this blog post will teach you how to write a python
script to relaunch every job stuck in your queue. (Bonus how to delete/cancel them
if needed too!)

## Relaunch

If you didn't know AWX has a solid _explicit_ API built in, and located at
`AWX_HOST/api/v2/`. You can poke around on it there, but the first thing you should
do is check out <AWX_HOST/api/v2/jobs>. You'll notice something that looks something
like this:
```json
HTTP 200 OK
Allow: GET, HEAD, OPTIONS
Content-Type: application/json
Vary: Accept
X-API-Node: awx-7bbbb66c54-zbrqs
X-API-Product-Name: AWX
X-API-Product-Version: 19.0.0
X-API-Time: 0.414s

{
    "count": 22418,
    "next": "/api/v2/jobs/?page=2",
    "previous": null,
    "results": [
        {
            "id": 4008,
            "type": "job",
            "url": "/api/v2/jobs/4008/",
            "related": {
                "created_by": "/api/v2/users/1/",
```

OK, lets take a look at some python code. The first thing you need to do is
authenticate with the API and get a good "count" of jobs. Here's the first step:

```python
import sys, requests, json
from requests.auth import HTTPBasicAuth

USER='admin'
PASSWORD='MY_AWESOME_PASSWORD'

AWX_HOST='https://awx.mydomain.net'
headers = {'content-type': 'application/json'}

res = requests.get(AWX_HOST + '/api/v2/jobs',
verify=True, auth=HTTPBasicAuth(USER, PASSWORD), headers=headers)

res_json = json.loads(res.text)

total_jobs=res_json.get("count")
print(total_jobs)
```

If this returns the same count as above, congrats, you now have a script to read
from your API! I realize there's probably better ways to do this, but hey it works.

Next we need to figure out how many pages you have, there's a couple of ways to figure
it out, but if you notice the pagination in the API web page, that'll short cut
your process.

Next you can remove the `print(total_jobs)` and past this in:
```python
total_pages = 40
# or do something like...
total_pages = total_jobs / 24
total_pages = int(total_pages)

for j in range(1,total_pages):
    print(AWX_HOST + '/api/v2/jobs' + '/?page=' + str(j))
    page_res = requests.get(AWX_HOST + '/api/v2/jobs' + '/?page=' + str(j), verify=True, auth=HTTPBasicAuth(USER, PASSWORD), headers=headers)
    page_res_json = json.loads(page_res.text)
    page_result_length=len(page_res_json["results"])
    for i in range(1,page_result_length):
        relaunch_url=str(page_res_json["results"][i]["related"]["relaunch"])
        print("relauch urls: " + relaunch_url)
        res = requests.post(AWX_HOST + relaunch_url,  verify=True, auth=HTTPBasicAuth(USER, PASSWORD), headers=headers)
```

There's a couple things going on here, but lets break it down. We walk through each
page with that first `for j in` line. We print out the pages so we know where we are,
then we do a `GET` request for that page. We load the output into the `json` parser
and figure out how long (how many jobs are) there.
Then with that in the `for i` line we walk through each one, looking for the
`relaunch` URL, and then post on it.

It walks through every job is the queue and relaunches it. (Now you can do specific
checks if you want, but this is very much a sledge hammer.)

All it is, is two `for` loops, but it's important to know that building off this
you can easily walk through your whole job queue.

## Deleting

At the writing of this, (2021-06-18) canceling _everything_ in your queue is also
something you might need to do. Unfortunately there isn't a button to select all
jobs so you need to build off that script above. Start with the `print(total_jobs)`
version and paste the following.
```python
for j in range(total_pages,0,-1):
    print(AWX_HOST + '/api/v2/jobs' + '/?page=' + str(j))
    page_res = requests.get(AWX_HOST + '/api/v2/jobs' + '/?page=' + str(j), verify=True, auth=HTTPBasicAuth(USER, PASSWORD), headers=headers)
    page_res_json = json.loads(page_res.text)
    page_result_length=len(page_res_json["results"])
    for i in range(1,page_result_length):
        print("id: " + str(page_res_json["results"][i]["id"]))
        res = requests.delete(AWX_HOST + '/api/v2/jobs/' + str(page_res_json["results"][i]["id"]),  verify=True, auth=HTTPBasicAuth(USER, PASSWORD), headers=headers)
        print(res.status_code)
```

Now there's a couple changes here, but it's worth talking about it too. You'll
notice that the `for j in range` line starts at the last page and moves back
to the first. This is because it seems that the "cascading" of canceled jobs
sometimes put AWX is a...weird state. If you delete all the dependant jobs
it seems to be happier. (I should mention you can't `DELETE` a running, so
you will need to cancel, which is just another `POST` to the `/cancel/` job
URL.)
We pull the same information off the team, then shove it back into our
`for i in range` and then do a `DELETE` request. We want a `status_code` of
`204` to say it successfully is taken. If you get anything else, you've 
done something wrong.


And that's it! It's just two `for` loops, but to unwind how to get to this
was hard to say the least.

Hopefully this'll help someone in the future.



[awx]: https://github.com/ansible/awx
