---
aliases:
- /develop/interact/search-and-query/advanced-concepts/spellcheck
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
description: Query spelling correction support
linkTitle: Spellchecking
title: Spellchecking
weight: 13
---

Query spelling correction provides suggestions for misspelled search terms. For example, the term 'reids' may be a misspelled version of 'redis'.

In such cases, and as of v1.4, RediSearch can be used for generating alternatives to misspelled query terms. A misspelled term is a full text term (i.e., a word) that is:

  1. Not a stop word
  2. Not in the index
  3. At least 3 characters long

The alternatives for a misspelled term are generated from the corpus of already-indexed terms and, optionally, one or more custom dictionaries. Alternatives become spelling suggestions based on their respective Levenshtein distances from the misspelled term. Each spelling suggestion is given a normalized score based on its occurrences in the index.

To obtain the spelling corrections for a query, refer to the documentation of the [`FT.SPELLCHECK`]({{< relref "commands/ft.spellcheck/" >}}) command.

## Custom dictionaries

A dictionary is a set of terms. Dictionaries can be added with terms, have terms deleted from them, and have their entire contents dumped using the [`FT.DICTADD`]({{< relref "commands/ft.dictadd/" >}}), [`FT.DICTDEL`]({{< relref "commands/ft.dictdel/" >}}) and [`FT.DICTDUMP`]({{< relref "commands/ft.dictdump/" >}}) commands, respectively.

Dictionaries can be used to modify the behavior of spelling corrections by including or excluding their contents from potential spelling correction suggestions.

When used for term inclusion, the terms in a dictionary can be provided as spelling suggestions regardless of their occurrence in the index. Scores of suggestions from inclusion dictionaries are always 0. 

Conversely, terms in an exclusion dictionary will never be returned as spelling alternatives.
