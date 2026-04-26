---
title: login.tpl
source: dle-news-docs
format: markdown
---

The output of this section is configured in the login.tpl file. This section configures the display of the visitor authorization panel on the website. The following tags can be used:

#### TAG: `{registration-link}`

Link to the visitor registration page

#### TAG: `{lostpassword-link}`

Link to the password recovery page

#### TAG: `{login}`

User login

#### TAG: `{logout-link}`

Link to log the user out on the website

#### TAG: `{admin-link}`

Link to the script admin panel

#### TAG: `{pm-link}`

Link to the personal messages page

#### TAG: `{new-pm}`

Number of new personal messages

#### TAG: `{all-pm}`

Total number of personal messages

#### TAG: `{favorite-count}`

Number of news items the user has added to bookmarks on the website

#### TAG: `{foto}`

Link to the user avatar.
`[admin-link]`
text
[/admin-link]

Outputs the text inside the tags if the user has access to the script admin panel

#### TAG: `{profile-link}`

Link to the user profile

#### TAG: `{stats-link}`

Link to website statistics

#### TAG: `{addnews-link}`

Link to the page for adding news on the website

#### TAG: `{favorites-link}`

Link to view the user's bookmarks

#### TAG: `{newposts-link}`

Link to view unread news for the user since their last visit to the website

#### TAG: `{group-icon}`

Outputs the user group icon

#### TAG: `{login-method}`

Depending on the authorization type configured in the script settings, outputs what the user must enter: login or email
`[vk]`
text
[/vk]

Output the enclosed text if authorization through the VK social network is enabled

#### TAG: `{vk_url}`

Outputs the URL link for authorization through the VK social network
`[odnoklassniki]`
text
[/odnoklassniki]

Output the enclosed text if authorization through the Odnoklassniki social network is enabled

#### TAG: `{odnoklassniki_url}`

Outputs the URL link for authorization through the Odnoklassniki social network
`[facebook]`
text
[/facebook]

Output the enclosed text if authorization through the Facebook social network is enabled

#### TAG: `{facebook_url}`

Outputs the URL link for authorization through the Facebook social network
`[google]`
text
[/google]

Output the enclosed text if authorization through the Google social network is enabled

#### TAG: `{google_url}`

Outputs the URL link for authorization through the Google social network
`[mailru]`
text
[/mailru]

Output the enclosed text if authorization through the Mail.ru social network is enabled

#### TAG: `{mailru_url}`

Outputs the URL link for authorization through the Mail.ru social network
`[yandex]`
text
[/yandex]

Output the enclosed text if authorization through the Yandex social network is enabled

#### TAG: `{yandex_url}`

Outputs the URL link for authorization through the Yandex social network
`[xfgiven_x]`
text
[/xfgiven_x]

Outputs the text enclosed in the tags if the additional field "x" has been specified in the user profile
`[xfnotgiven_x]`
text [/xfnotgiven_x]

Outputs the text specified in these tags if the user profile additional field was not specified, where X is the name of the user profile additional field
`[xfvalue_x]`
Outputs the value of the additional field "x", where "x" is the name of the additional field

#### TAG: `{group}`

Outputs the name of the website user group the user currently belongs to