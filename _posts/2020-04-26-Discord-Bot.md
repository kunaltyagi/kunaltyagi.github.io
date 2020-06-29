---
layout:            post
title:             "Making a discord bot (and why?)"
date:              2020-04-26 18:25:00 +0900
tags:              Discord Python
category:          Walkthrough
author:            ktyagi
---


## The Setup
Sometimes in a conversation, people wish for unicorns and rainbows. Sometimes, a simple
python script suffices, and sometimes that script can mutate into a bot.

PCL went through a tumultous phase, with the mailing list going down, and Gitter was found
to be lacking for community interactions and with GSoC right around the corner, Discord
was tried and found to be a nice complement to GitHub issues and wiki.

During one conversation, people were lamenting the large number of open issues, many of
which could be closed if someone took a look to check if they were still valid after months
of inactivity. The large number of issues was daunting and the idea of a '*Issues of the
week*' was suggested.

## The Conflict

One person picking 5 random issues and posting for others to update, add-info or close
was a potential option, but I was concerned that
* if left to humans, it'd be ngelected
* having a bus-factor of 1 has bit PCL in the past

I decided to create a bot to send 5 issues daily. How hard can that be? GitHub has an API.
Someone must have made a wrapper for it. Discord is famous for bots. There must be a wrapper
for Discord API.

Filtering the possibilities using my skills in parseltongue (called Python by the plebs),
I narrowed the field to 2 possible libraries:
* For GitHub: [PyGitHub](https://github.com/PyGithub/PyGithub)
* For Discord: [discord.py](https://github.com/Rapptz/discord.py)

I didn't see an async interface in PyGitHub, which led me to the
[GitHub API](https://api.github.com/). Using a non-asynchronous interface could result in
unwanted blocking for the discord.py's async implementation.

Getting issues from GitHub is pretty straightforward. Using the very popular `requests`
library, getting issues from the PCL repo is as:
```python
url = "https://api.github.com/search/issues?q=is:issue+repo:PointCloudLibrary/pcl"
response = requests.get(url)

# raise exception if status is more than 200 OK
response.raise_for_status()

# GitHub returns data as JSON
data = response.json()
issues = data["items"]
num_issues = data["total_count"]
```
GitHub also implements pagination, which means the number of issues returned per request is
limited (and hits a max of 100 per page). In order to fix this, we can modify the url to
include
* a high number of issues per page (to reduce total number of requests). GitHub's maximum
  allowed limit is 100
* page number, starting with 1 till we get total issues recieved (all `data["items"]`) equal
  to total issues reported (`data["total_count"]`)

```python
url = "https://api.github.com/search/issues?q=is:issue+repo:PointCloudLibrary/pcl"
issues = []
page = 1

# Loop till we get all issues
while True:
    response = requests.get(f"{url}&page={page}&per_page=100")

    # raise exception if status is more than 200 OK
    response.raise_for_status()

    # GitHub returns data as JSON
    data = response.json()
    total_count = data["total_count"]
    issues.extend(data["items"])

    # prepare to recieve next page, if required
    page += 1
    if len(issues) == total_count:
        break
```
Notice the use of format-string `f"{url}&page={page}"` which is a (relatively) new feature,
and saves having to use the alternatives, the second of which looks almost the same:
* string concatenation (`url + '&page=' + page`)
* string formatting (`'{url}&page={page}'.format(url=url, page=page)`)

Filtering issues with a certain label adds more complication to the url string. GitHub has
a format for including and excluding labels. If the label has a space, it needs to be
inside double quotes, and no, single quotes don't work.
* Including issues: `[f'label:"{gh_encode(x)}"' for x in include_labels]`
* Excluding labels: `[f'-label:"{gh_encode(x)}"' for x in exclude_labels]`

`gh_encode` is needed because the label is encoded to be valid value in the url. A simple
implementation would look like:
```python
def gh_encode(x):
    from urllib.parse import quote_plus
    # GitHub wants values to be added with a `+` and considers `"` to be safe
    return quote_plus(x, safe='"')
```


Getting issues with async adds a slight
twist.


This is a [blog template](https://github.com/jwillmer/jekyllDecent) for a static site generator named [Jekyll](https://jekyllrb.com/docs/home/) based on a [Ghost](https://ghost.org) template named [Decent](https://github.com/serenader2014/decent). 
If you like to see the theme in production have a look at [jwillmer.de](http://jwillmer.de).


## Screenshots

<div class="album">

<figure>
<img src="{{ "/media/img/2016-06-08-Readme-front-page-previewe.jpg" | absolute_url }}" />
<figcaption>Frontpage</figcaption>
</figure>

<figure>
<img src="{{ "/media/img/2016-06-08-Readme-post-preview.jpg" | absolute_url }}" />
<figcaption>Post</figcaption>
</figure>

<figure>
<img src="{{ "/media/img/2016-06-08-Readme-content-preview.jpg" | absolute_url }}" />
<figcaption>Demo</figcaption>
</figure>

<figure>
<img src="{{ "/media/img/2016-06-08-Readme-about-preview.jpg" | absolute_url }}" />
<figcaption>About Page</figcaption>
</figure>

</div>


## Installation

- To generate/host the blog you need to [install Jekyll](https://jekyllrb.com/docs/installation/). 
- For the plugins (github-pages) you need to install [Bundler](http://bundler.io/): `gem install bundler`
   - Open a command prompt and install the plugins (github-pages): `bundle install`
   - On Windows you can get an exception if you have not the latest rubygems. To solve it you get the latest rubygems or you can read [How to install Jekyll and pages-gem on Windows (x64)](http://jwillmer.de/blog/tutorial/how-to-install-jekyll-and-pages-gem-on-windows-10-x46) or you just remove the plugins by deleting the `Gemfile` and delete all `gems:` from the `_config.yml`. 

### Build

- To build the static site you can use the generated site folder that Jekyll creates when you use `jekyll serve` or you can build it explicitly with `jekyll build`.
- To [auto refresh your browser on changes](https://github.com/awood/hawkins) run jekyll with the following command `jekyll liveserve`
- If you like to use GitHub to host your blog you can [fork this project](https://github.com/jwillmer/jekyllDecent) and publish the code to `gh-pages`. GitHub has jekyll included and will generate the site for you.


## User Content

Blogposts can be written in [Markdown](https://de.wikipedia.org/wiki/Markdown). 

- The folder for blog content is `_posts`
- For author details you need to modify `_data/authors.yml`
- For cv details you need to modify `_data/cv.yml` 
- If you change the author in `_data` you need to change the `author` attribute in `_posts` and `_pages` as well 
- For site properties (like the name) you need to modify `_config.yml` and `robots.txt`

After modifying `*.yml` files you need to restart jekyll to take effect.

### YAML Features

Following (additional) features are supported in the header ([YAML Front Matter](https://jekyllrb.com/docs/frontmatter/)) of each post. A detailed description can be found in the [YAML Custom Features post]({{ "/blog/features/YAML-Features" | absolute_url }}).

```bash
---
title:         Example      #Page/post title
author:        jwillmer     #Page/post author
cover:         /image.jpg   #Optional: Posibillity to change cover on a post/page
redirect_from: /foo         #Optional: Redirect url
visible:       false        #Optional: Hide page from listing in the menu.
weight:        5            #Optional: Influence sorting of pages in the menu
menutitle:     Offline      #Optional: Use a secondary name in the menu/post list
tags:          hallo welt   #Optional: Will be displayed as tags and as keywords in posts
keywords:      hallo welt   #Optional: Will add keywords to a page
language:      en           #Optional: Will set a specific language of the page
comments:      false        #Optional: Will enable/disable comments in your post 
math:          false        #Optional: Will enable math formulas
citation:      [..]         #Optional: Additional meta tags for scholar articles
---
```

## License

The theme is released under **[The MIT License (MIT)](https://github.com/jwillmer/jekyllDecent/blob/gh-pages/LICENSE)**.

    The MIT License (MIT)

    Copyright (c) 2016 Jens Willmer

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:
    
    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.
    
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.    
