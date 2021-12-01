# CI Jobs

## Layerfile runner objects
Each time you look at the dashboard for a run, you're looking at a collection of layerfile runners.
Each layerfile runner is created by a specific Layerfile, or by a Layerfile being retried.

*CI jobs are a collection of Layerfile runners!*

## Example schema

```json
{
    "repository_name": "webappio-example",
    "id": 1,
    "status": "SEE_RUNNERS",
    "status_change_time": "2000-01-01 01:23:45.123456 +0000 UTC",
    "calculated_status": "SUCCEEDED",
    "commit_sha": "9abc2ac68d52afe1a5a3fbc724d031af5a397204",
    "branch": "master",
    "merge_branch": "master",
    "run_start_reason": "Manual run created by API",
    "run_start_reason_url": ""
}
```

## status vs. calculated_status

CI jobs themselves maintain an internal status which is either:

- `NEW` for pending jobs
- `ERROR` for jobs that have had some error while initializing (e.g., invalid layerfiles)
- `STARTING_RUNNERS` for jobs which are in the process of starting runners  
- `SEE_RUNNERS` for jobs which have started runners. This status means you must look at the runners themselves to understand the status.


CI jobs also contain a status which is the calculated status of the whole job, looking at the runners.
The calculated status is usually what you'll use to determine the status of an entire run.

The `calculated_status` of a CI job is one of the following:

- `NEW` for pending jobs
- `JOB_ERROR` for jobs that have had an internal while initializing (e.g., internal errors in webapp.io occurred when starting this run)
- `BAD_LAYERFILE` for jobs that had an invalid Layerfile
- `STARTING_RUNNERS` means that runners are in the process of being started for this job.
- `FAILURE` means that at least one runner failed (some instruction for that runner is red) - again, there can be multiple Layerfiles per repository and if either fails.
- `RUNNER_CANCELLED` means that a user manually cancelled at least one runner
- `RUNNER_ERROR` means that at least one runner had an internal error while running
- `RUNNING` means that the runner itself is running
- `SUCCEEDED` means that the runner succeeded


## Get most recent CI job for a repository matching a search query

It's often useful to get the most recent CI job given specific filters. This API endpoint allows for that.

<language-tabs>

```ruby
require 'faraday'
require 'json'

# get the most recent failing jobs
res = Faraday.get 'https://webapp.io/api/v1/run/repo_name/search?search=status%3Asucceeded&token=(your api key)'

res = JSON.parse(res.body)
```

```python
import requests

# get the most recent failing job
my_token="(your API key)"
res = requests.get(
    'https://webapp.io/api/v1/run/repo_name/search', 
    params={
        "search": "status:succeeded",
        "token": my_token
    }, 
).json()
```

```shell
# get the most recent failing job
my_token="(your API key)"
curl 'https://webapp.io/api/v1/run/repo_name/search?search=status%3Asucceeded&token=${my_token}' | python -m json.tool
```

```javascript
let apiKey="(your API key)"
fetch(
    'https://webapp.io/api/v1/run/repo_name/search?search=status%3Asucceeded&token='+apiKey,
).then(res => res.json()).then(json => console.log(json))
```

</language-tabs>

Output:


```json
{
    "status": "ok",
    "job": {
        "repository_name": "webappio-example",
        "id": 3,
        "status": "SEE_RUNNERS",
        "status_change_time": "2000-01-01 01:23:45.123456 +0000 UTC",
        "calculated_status": "SUCCEEDED",
        "commit_sha": "9abc2ac68d52afe1a5a3fbc724d031af5a397204",
        "branch": "master",
        "merge_branch": "master",
        "run_start_reason": "Manual run created by API",
        "run_start_reason_url": ""
    }
}
```

-- or --
```json
{
    "error": "There are no runs in layerdemo/webappio-example matching the given filters.",
    "status": "error"
}
```

## Search Filters
The search query for /search and the main dashboard's searchbar are the same, and can include:

1. natural text
2. "quoted exact text"
3. -minus -some -terms that should not appear in the author, commit body, or title of the commit.
4. repo:(the repository name) to match an exact repository name
5. branch:(the branch name) to match an exact (case sensitive) branch name
6. id:(the id) to match an exact run id
7. status:(status) to match a [calculated status](#status-vs-calculated-status)
8. `running` to match a run which is running
9. `done` to match a run which is not running
10. `failed` to match a run which has suffered a failure or error

##### Example query
`text to find in commit body "exact text" -layer -ci repo:layer-example-repo branch:main id:10 status:running done running failed`

## Create a CI job for a given repository

Sometimes it's useful to manually start webapp.io runs (for, e.g., deploying)
This endpoint lets you do that.

The input to the POST is optional, but can contain:

- `branch=master` to check out the "master" branch. If omitted, we use "master"
- `ref=9abc2ac68d52afe1a5a3fbc724d031af5a397204` to check out a specific commit. If omitted, we use "origin/master"
- `accept_buttons=true` to accept any BUTTON instructions in the job (i.e., to deploy automatically)
- `extra` - extra data exposed in the run as API_EXTRA, useful for passing arbitrary data in

<language-tabs>
```ruby
require 'faraday'
require 'json'

res = Faraday.post(
    'https://webapp.io/api/v1/run/repo_name?token=(your api key)',
    {
        branch: 'master',
        ref: '9abc2ac68d52afe1a5a3fbc724d031af5a397204',
        accept_buttons: 'true',
        extra: 'some extra data exposed as API_EXTRA variable',
    }.to_json
)

res = JSON.parse(res.body)
```

```python
import requests

my_token="(your API key)"
res = requests.post(
    f'https://webapp.io/api/v1/run/repo_name',
    params={"token": my_token}, 
    json={
        'branch': 'master',
        'ref': '9abc2ac68d52afe1a5a3fbc724d031af5a397204',
        'accept_buttons': 'true',
        'extra': 'some extra data exposed as API_EXTRA variable',
    },
).json()
```

```shell
my_token="(your API key)"
curl -X POST \
    -H 'Content-Type: application/json' \
    -d '{"branch": "master", \
         "ref": "9abc2ac68d52afe1a5a3fbc724d031af5a397204", \
         "accept_buttons": "true", \
         "extra": "some extra data exposed as API_EXTRA variable" \
        }' \
    "https://webapp.io/api/v1/run/repo_name?token=${my_token}"
```

```javascript
let apiKey="(your API key)"
fetch(
    'https://webapp.io/api/v1/run/repo_name?token='+apiKey,
    {
        method: "POST",
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            'branch': 'master',
            'ref': '9abc2ac68d52afe1a5a3fbc724d031af5a397204',
            'accept_buttons': 'true',
            'extra': 'some extra data exposed as API_EXTRA variable',
        }),
    }
).then(res => res.json()).then(json => console.log(json))
```
</language-tabs>

Output:

```json
{
    "status": "ok",
    "job": {
        "repository_name": "webappio-example",
        "id": 1,
        "status": "NEW",
        "status_change_time": "2020-05-05 05:11:44.17955 +0000 UTC",
        "calculated_status": "NEW",
        "commit_sha": "9abc2ac68d52afe1a5a3fbc724d031af5a397204",
        "branch": "master",
        "merge_branch": "master",
        "run_start_reason": "Manual run created by API",
        "run_start_reason_url": ""
    },
    "url": "http://webapp.io/jobs/github/distributed-containers-inc/webappio-example/1"
}

```

`status` can be one of `"ok"` or `"error"`. If the latter exists, an `"error"` value will be included with an explanation.
