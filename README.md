# Hugo SEO Module

Handles all of your SEO needs:

- Meta tags
- Facebook opengraph meta tags
- Twitter card tags
- Sitemap
- XML feeds
- Robots.txt
- JSON-LD breadcrumbs
- JSON-LD article (see config below to set .types for article)

## Requirements

Requirements:

- Go 1.14
- Hugo 0.109.0

## Installation

If not already, [init](https://gohugo.io/hugo-modules/use-modules/#initialize-a-new-module) your project as Hugo Module:

```terminal
$: hugo mod init github.com/user/project_name
```

Configure your project's module to import this module:

```yaml
# config.yaml
module:
  imports:
    - path: github.com/future-wd/hugo-seo
```

Get the module with `hugo mod get -u github.com/future-wd/hugo-seo`

## Usage

Drop the following in your websites HEAD

```go-html-template
{{ partial "hugo-seo/tags.html" . }}
```

The above partials, will look for content information and build an Data object to be printed in SEO tags (og, twitter card etc...).

> Note:
>
> By default the module creates a `<title>` tag. This can be disabled with configuration, see below.

## Site Config

Settings are added to the project's params under the `seo` map as shown below.

Defaults have been shown.

```yaml
# config.yaml
title: Set your site's title here
enableRobotsTXT: true # this is set for you, but don't set it to false in your site!
params:
  description: Default SEO description for pages without one.
  seo:
    generate: # choose which tags are generated
      title: true
      meta: true
      twitter: true
      og: true
      jsonld:
        article: true
        breadcrumbs: true
    title_tag:
      separator: "|"
      home_text: "" # this text is added to the title tag on home page
    og_article_types: [post, posts, blog, news, article, articles, event, events, course, courses]
    jsonld_article_types: [article, articles]
    jsonld_news_article_types: [news, updates]
    jsonld_blog_posting_types: [post, posts, blog]
    # page or site
    image: # set default here, page override can be set. 
    private: false # set here or per page to modify robots meta, remove sitemap, robots and rss listing for page(s)
```

## Page config (markdown)

```yaml
seo:
  title: # page title override for title tag, og, twitter and json-ld
  description: # override description/summary
  canonical: # add page ref to override .Permalink for canonical
```

### Site Title

Set the sites title in the site config.

### enableRobotsTXT

This has already been set to true in the modules config. DO NOT have `enableRobotsTXT: false` in your site config, because it will override the SEO module.

### params.description

Set a default description that will be used if the page does not have a description or summary.

### params.seo.generate

Here you an disable the generation of different tag types. This can also be set at a page level if you wish to disable a tag just for one page e.g. params.seo.generate.jsonld.article: false if you are custom generating an event json-ld script for that page.

The module generates a title tag. If you can't disable your theme's title, change the config to false.

### params.seo.title_separator

The home page has `<title>{site title}</title>`

Other pages have `<title>{page title} | { site title }</title>`

You can change the separator from "|" to another character e.g. "-"

### params.seo.site_name

You can set a site title overide for use in the open graph tag.

### params.seo.og_article_types

The types in the array will be recognised as articles for opengraph tag generation.

### params.seo.jsonld*

JSON-LD article has three different types, according to these arrays.

### params.seo.image

Provide a default image should a page not have one. It can be a global resource (assets dir) or static file (static dir) path.

#### params.seo.private

If site config has private: true set, the following will take place:

- `<meta name="robots" content="noindex, nofollow" >`
- robots.txt: `Disallow: *`, no link to sitemap
- RSS feeds: disabled
- Sitemap: empty file

You can set seo.private: true at a site level to make every page public.

This can be overridden at a page level e.g.

site:  params.seo.private: false

AND

```markdown
---
# content/private/index.md (see below)
seo:
  private: true
---
```

Will result in the following output:

- `<meta name="robots" content="noindex, nofollow" >` for 1 page and the rest `<meta name="robots" content="index, follow" >`
- robots.txt: `Disallow: /private/index.html`, sitemap linked
- RSS feeds: /private/index.html omitted
- Sitemap: /private/index.html omitted

### Front Matter

The module uses title, description, summary and type from frontmatter.

Additionally there is a `seo` map for overrides specific to the module.

Add any of the configurations to override your site configuration.

```markdown
---
title: Page title
description: Takes precedence over summary
summary: Page summary
type: set the type, the module may categorise the page as an article or event.
seo:
  generate: # set to false to disable a tag
    title: true
    meta: true
    twitter: true
    og: true
    jsonld:
      article: true
      breadcrumbs: true
  title: Overrides the page title
  image: path to page resource, or static file
  description: Overrides page description
  canonical: path to actual page if this is a duplicate
---
```

> Note:
>
> Don't copy and paste in this whole list, as it may override many of your defaults in site config!

## Credits

Sean Emerson - Future Web Design

Originally forked from tnd-seo by [thenewDynamic](https://www.thenewdynamic.com).

## Errors, Feature requests and Contributions

Log an error or feature request as an issue.

PR's are welcome.


### Params

The following parameters are in the current context when creating a custom json-ld type.

Variable are in camelCase form if matching a json-ld property. Otherwise they take the form of their purpose e.g. twitter or og variable name.

| Variable          | Description | Source |
| ----------------- | ----------- | ------ |
| .page             | access to the current page context | .Page |
| site              | hugo function for accessing .Site context | reference only |
| .date_published    | ISO datestamp publish time | .PublishDate, .Date |
| .date_modified     | ISO datestamp of modified time |.Lastmod |
| .description      | Page description plain text | .Params.seo.description, .Description, .Summary, site.Params.seo.description, site.Params.description |
| .title            | Page title | .Params.seo.title, .Title |
| .site_title        | Site Title | site.Title, site.Params.title |
| .image            | absURL of page image | .Params.seo.image, index (.Params.images) 0, .Params.image, site.Params.seo.image (static file, page resource or global resource) |
| .image_height      | height of image | image variable above |
| .image_width       | width of image | image variable above |
| .image_type        | mime type of image if page or global resource | image variable above |
| .private          | boolean | .Params.seo.private, site.Params.seo.private |
| .locale           | site language code | .Lang |
| .canonical        | canonical absUrl  | page: .Params.seo.canonical, .Permalink |
| -                 | -                 | node: .Paginator.URL |
| .next             | next page absURL if node | .Paginator.Next.URL |
| .prev             | prev page absURL if node | .Paginator.Prev.URL |
| .alternative_output_formats | alternative output formats array | .alternative_output_formats |
| .twitter_card     | summary or summary_large_image | if image has been set |
| .twitter_site     | site twitter handle (with @) | site.Social.twitter (no @) |
| .twitter_creator  | page author twitter handle (with @) | .twitter in .author or first .authors (no @) |
| .og_type           | og page type | website or if .Section is in og_article_types, article |
| .audio            | page audio clip absURL | .Params.audio |
| .video            | page video clip absURL | index (.Params.videos) 0, .Params.video |
| .see_also         | array of related pages | first 6 related pages, then first 6 pages by date in same section |
| .locale_alternate | array of translations of page | .Lang for each of .Translations  |
| .jsonld_type       | type of page for jsonld | if in .Section: jsonld_article_types > article, jsonld_news_article_types > newsArticle, jsonld_blog_posting_types > blogPosting |
| .article_body      | page body plain text | .Plain |
| .authors          | slice of authors | .Params.authors, .Params.author (provide name and url, optionally twitter) |
| .breadcrumbs      | breadcrumbs array | current page and parents |
