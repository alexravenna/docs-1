---
acl_categories:
- '@read'
- '@search'
arguments:
- name: index
  type: string
- name: cursor_id
  type: integer
- name: read size
  optional: true
  token: COUNT
  type: integer
categories:
- docs
- develop
- stack
- oss
- rs
- rc
- oss
- kubernetes
- clients
complexity: O(1)
description: Reads from a cursor
group: search
hidden: false
linkTitle: FT.CURSOR READ
module: Search
since: 1.1.0
stack_path: docs/interact/search-and-query
summary: Reads from a cursor
syntax: 'FT.CURSOR READ index cursor_id [COUNT read_size]

  '
syntax_fmt: "FT.CURSOR READ index cursor_id [COUNT\_read size]"
syntax_str: "cursor_id [COUNT\_read size]"
title: FT.CURSOR READ
---

Read next results from an existing cursor

[Examples](#examples)

See [Cursor API]({{< relref "develop/ai/search-and-query/advanced-concepts/aggregations#cursor-api" >}}) for more details.

## Required arguments

<details open>
<summary><code>index</code></summary>

is index name.
</details>

<details open>
<summary><code>cursor_id</code></summary>

is id of the cursor.
</details>

<details open>
<summary><code>[COUNT read_size]</code></summary>

is number of results to read. This parameter overrides `COUNT` specified in [`FT.AGGREGATE`]({{< relref "commands/ft.aggregate/" >}}).
</details>

## Examples

<details open>
<summary><b>Read next results from a cursor</b></summary>

{{< highlight bash >}}
127.0.0.1:6379> FT.CURSOR READ idx 342459320 COUNT 50
{{< / highlight >}}
</details>

## Return information

{{< multitabs id="ft-cursor-read-return-info" 
    tab1="RESP2" 
    tab2="RESP3" >}}

One of the following:
* [Array]({{< relref "/develop/reference/protocol-spec#arrays" >}}) with search results and metadata.
* [Simple error reply]({{< relref "/develop/reference/protocol-spec#simple-errors" >}}) in these cases: cursor not found.

-tab-sep-

One of the following:
* [Map]({{< relref "/develop/reference/protocol-spec#maps" >}}) with structured search results and metadata.
* [Simple error reply]({{< relref "/develop/reference/protocol-spec#simple-errors" >}}) in these cases: cursor not found.

{{< /multitabs >}}

## See also

[`FT.CURSOR DEL`]({{< relref "commands/ft.cursor-del/" >}}) | [`FT.AGGREGATE`]({{< relref "commands/ft.aggregate/" >}})

## Related topics

[RediSearch]({{< relref "/develop/ai/search-and-query/" >}})
