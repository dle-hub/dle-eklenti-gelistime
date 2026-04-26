---
title: search.tpl
source: dle-news-docs
format: markdown
---

The output of this section is configured in the search.tpl file. This section configures the templates responsible for displaying the website search form:

#### TAG: `{searchtable}`

Search form displayed as a table

#### TAG: `{searchmsg}`

Outputs the search result message
`[searchmsg]`
text
[/searchmsg]

Outputs text if a search has been performed
`[simple-search]`
text
[/simple-search]

Outputs the text enclosed in the tags if a simple search is being performed
`[extended-search]`
text
[/extended-search]

Outputs the text enclosed in the tags if a search with extended parameters is being performed

#### TAG: `{searchfield}`

Outputs the field for entering the text to be searched for

#### TAG: `{word-option}`

Outputs the checkbox for the search text option (exact match of all words or not)

#### TAG: `{search-area}`

Outputs the selection of the search area on the website

#### TAG: `{userfield}`

Outputs the field for entering the publication author

#### TAG: `{user-option}`

Outputs the checkbox for the author search option (exact match of all words or not)

#### TAG: `{news-option}`

Outputs the selection of comment-related parameters for news items

#### TAG: `{comments-num}`

Outputs the field for entering the number of comments on news items

#### TAG: `{date-option}`

Outputs the selection of date-related parameters for news items

#### TAG: `{date-beforeafter}`

Outputs the choice of whether the found news items should be newer or older than the specified date

#### TAG: `{sort-option}`

Outputs the selection of sorting parameters for found results

#### TAG: `{order-option}`

Outputs the selection of sort order (ascending or descending)

#### TAG: `{view-option}`

Outputs the selection of the display mode for found results

#### TAG: `{category-option}`

Outputs the selection of categories in which the search should be performed