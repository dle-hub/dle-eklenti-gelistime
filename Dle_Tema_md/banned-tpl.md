---
title: banned.tpl
source: dle-news-docs
format: markdown
---

The output of this section is configured in the banned.tpl file. This section configures the templates responsible for displaying information that the visitor is denied access to the website:

#### TAG: `{description}`

Outputs the reason why access to the website was blocked for the visitor

#### TAG: `{end}`

Outputs the blocking end date if it was set
`[banned-from]`
text
[/banned-from]

Output the enclosed text if the block was assigned by an administrator

#### TAG: `{banned-from}`

Outputs the login of the user who set the website block.