# dstask Plugin Spec Proposal

## Motivation

A dstask entry looks like this, currently. Some of these fields are reserved 
for future use.

```yaml
summary: Do something important
notes: ""
tags:
- personal
project: ""
priority: P2
delegatedto: ""
subtasks: []
dependencies: []
created: 2021-11-11T15:06:17.221166594-05:00
resolved: 0001-01-01T00:00:00Z
due: 0001-01-01T00:00:00Z
```

_Note: the "zero value" of a Go time.Time type looks like year 1, C.E._


We've received requests to add fields, and change the processing of fields.
We've also received requests around changing sorting, color, and other aspects
of how things render.

While these features are easy to implement, some of them introduce breaking
changes, and change behavior that many users are already happy with.

Dstask can be scripted, of course, if you can glue together shell code and `jq`,
but this isn't an option for many users.

## Role of Plugins

Plugins will let users do the following

* Work with custom fields that don't exist today
* Add new subcommands to the `dstask` command line
* Implement pre-commit processing pipelines that can change task data before
  it hits the database
* Implement alternative ways of querying and selecting from the dstask databse

## Changes to dstask Core

1. Add a `meta` field of type `map[string]interface{}` to the Task core
2. Add a new environment variable `DSTASK_PLUGINS` which must be present to opt
   in to plugins.
3. Add "hook" type code paths at various points in the existing code (TODO be more specific)
   * Sometime before a task is written or committed (Task Mutating Plugins)
   * Immediately before writing to stdout (Display Plugins)
   * An alternative codepath that deserializes an existing task list (Query Plugin)

## Plugin Types

### Task Mutating Plugins

```
read only: false
```

### Display Plugins

```
read only: true
```

### Query Plugins

```
read only: true
creates subcommand: true
```


## Example setup

I've set the following variable in my shell. It points to a directory, where
dstask will look for plugins.

```
export DSTASK_PLUGINS=$HOME/.local/lib/dstask
```

Every file in that directory will be a plugin iff

* it is an executable, or a symlink to an executable
* the filename is of the form `dstask-{command}-{pluginName}`

The naming of the executable has meaning, and declares how it should be
used by dstask.

Consider this example, which adds a pre-commit hook for add, a built in command.

```
dstask-add-repoInfo
```

This plugin will accept (something like) the following json on stdin.

<details>
	<summary>see json</summary>

```json
  {
    "uuid": "6d7fb61b-628e-4a87-8ed3-8d61081c4d2e",
    "status": "pending",
    "summary": "Deploy version 2 to server",
    "notes": "",
    "tags": [
      "work"
    ],
    "project": "",
    "priority": "P2",
    "created": "2021-11-05T14:59:10.781660165-04:00",
    "resolved": "0001-01-01T00:00:00Z",
    "due": "0001-01-01T00:00:00Z",
    "meta": {}
  }
```

Note our new "meta" field.

</details>


The plugin itself is this python script.

```py
#!/usr/bin/env python3
import json, sys

def get_git_repo():
	pass  # TODO

# load stdin as json, and extract tags
task = json.load(sys.stdin)
tags = task['tags']

# we only modify tasks with the tag "work"
if work in tags:
	info = get_git_repo()
	# add subfield of meta: a 'repo' field, with some string value
	task['meta']['repo'] = info['origin']

# write the (maybe modified) task to stdout
print(json.dumps(task))

```

The modified task now has git repository information embedded in its
"meta" field.

<details>
	<summary>See json</summary>

```json
  {
    "uuid": "6d7fb61b-628e-4a87-8ed3-8d61081c4d2e",
    "status": "pending",
    "summary": "Deploy version 2 to server",
    "notes": "",
    "tags": [
      "work"
    ],
    "project": "",
    "priority": "P2",
    "created": "2021-11-05T14:59:10.781660165-04:00",
    "resolved": "0001-01-01T00:00:00Z",
    "due": "0001-01-01T00:00:00Z",
    "meta": {
    	"repo": "git@github.com:coolgirl234123/server.git"
    }
  }
```

</details>


## Plugin Evaluation

We could allow multiple plugins for the same command.

```
dstask-add-recipeInfo
dstask-add-repoInfo
```

We can sort them lexically, and evaluate them in order.


## Plugin Security

Let's take reasonable steps to guard against badly written
plugins. There's only so much we can do to guard against
actively _malicious_ plugins, but we should explore those
things for various systems.

See: systemd-run --scope for newer linuxes, for example.

At a minimum, we should have a reasonable default timeout.
A well-behaved plugin should run _very_ quickly.
