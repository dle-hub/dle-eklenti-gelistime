---
title: poll.tpl
source: dle-news-docs
format: markdown
---

The output of this section is configured in the poll.tpl file. This section configures the display of a poll for a published news item.

#### TAG: `{title}`

Poll title

#### TAG: `{question}`

Question

#### TAG: `{votes}`

Total number of votes

#### TAG: `{list}`

List of answer options
`[voted]`
text
[/voted]

Output the text in the tags if the user has already voted in the poll; otherwise it is hidden
`[not-voted]`
text
[/not-voted]

Output the text in the tags if the user has not voted in the poll yet; otherwise it is hidden
`[closed]`
text
[/closed]

Output the enclosed text if the poll has been closed for voting
`[not-closed]`
text
[/not-closed]

Output the enclosed text if the poll is open for website visitors to vote

#### TAG: `{close-date}`

Outputs the poll closing date if it has been closed