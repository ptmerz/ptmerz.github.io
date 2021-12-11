---
title: "Enabling blog comments using `giscus`"
tags: ["Minimal Mistakes", "Jekyll", "Comments", "giscus"]
category: "Blogging platform"
---

Minimal Mistakes offers
[built-in support](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#comments)
for commenting via Disqus, Discourse, Facebook, utterances, giscus and Staticman. The first
three are external services. I didn't pursue these options further, because I would prefer to
avoid external trackers and heavy assets on my website. Staticman allows adding comments as
pushes or pull requests to the static site, making them effectively part of the static site.
This would certainly be the most light-weight option, but the setup process and especially
the spam handling are a bit intimidating.

utterances and giscus leverage GitHub functionality to provide comments. Both are open source
and promise to have no tracking, no ads, and be always free. utterances (mis)uses
GitHub issues on the repository for the comments, while giscus uses the new GitHub discussions
functionality. Since the latter sounds more like using functionality how it was meant to be
used and less like a hack, I decided to go for giscus.

The Minimal Mistakes
[documentation](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#giscus-comments)
does a great job at explaining the steps, which roughly consist of
1. Turn on the "Discussions" feature of the repository under Settings > Options > Features.
2. Install the [giscus](https://github.com/apps/giscus) app in the repository.
3. Head to [https://giscus.app](https://giscus.app) and fill out the details. Most of it is
   self-explanatory and for the rest I went with the default choice hoping for the best.
   Specifically, I wasn't sure about the "Page ↔️ Discussions Mapping", so I left the default
   "Discussion title contains page `pathname`". I picked the "Announcements" category and chose
   to enable reactions. These settings are populating a JavaScript snippet that appears at the 
   bottom of the page. Since Minimal Mistakes has built-in functionality for giscus, we will 
   actually not copy that snippet, but use the settings in the Minimal Mistakes configuration file.
4. Adapt the Minimal Mistakes `_config.yml`: Make sure that the `repository` tag is set, set the
   `provider` in the `comments` section to `"giscus"`, and fill the variables in the `giscus`
   section by using the values in the JavaScript snippet.
5. Finally, set `comments: true` in the YAML front matter of the posts. This can either be done
   on a per-post-basis, by adding this in the front matter of the posts that should have comments,
   or globally by changing the default in the `_config.yml` file for the posts category.

Having done all that, I pushed and reloaded the page and - nothing showed up! It turns out that
Minimal Mistakes only turns comments on when Jekyll is run in a `production` environment, while 
Jekyll is in a `development` environment by default
([reference](https://jekyllrb.com/docs/configuration/environments/)). The GitHub action I'm using
to build and deploy the Jekyll site was running Jekyll in a `development` environment. Fixing that
did the trick.

You can see the results of my efforts under this post! I am actually really happy with how that
looks and feels. I think that a lot of people that could want to interact with these posts will
have a GitHub account, so using that account to comment seems like a very low barrier. If people
prefer not to grant giscus the required GitHub OAuth authorization, they can post directly on the
GitHub Discussions. The comments support the same rich Markdown and reactions as is being used
all over GitHub, and the option to add post reactions is a nice little addition. Finally, I also
think that the looks fit well with the rest of the site layout.
