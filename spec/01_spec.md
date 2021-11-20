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
   * Sometime before a task is committed (Task Mutating Plugins)
   * Immediately before writing to stdout (Display Plugins)
   * An alternative codepath that deserializes an existing task list (Query Plugin)


## Plugin Types

### Task Mutating Plugins

| read only | false |

### Display Plugins

| read only | true |

### Query Plugins

| read only | true |


