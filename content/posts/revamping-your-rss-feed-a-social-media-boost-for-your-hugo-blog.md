---
title: Revamping Your RSS Feed - A Social Media Boost for Your Hugo Blog
date: 2024-01-06
description: "Hey peeps! Ever felt like your blog posts were missing a little extra flair when shared on social media? Well, you're not alone. In my quest to supercharge my Twitter and Mastodon game, I stumbled upon a nifty solution involving customizing my RSS template for Hugo. And let me tell you, it's been a game-changer."
image: images/posts/revamping-your-rss-feed-a-social-media-boost-for-your-hugo-blog-banner.png
imageAltAttribute: A bad AI generated image of a cyber dinosaur surfing through cyberspace.
tags:
   - hugo
   - blogumentation
---

Hey peeps! Ever felt like your blog posts were missing a little extra flair when shared on social media? Well, you're not alone. In my quest to supercharge my Twitter and Mastodon game, I stumbled upon a nifty solution involving customizing my RSS template for Hugo. And let me tell you, it's been a game-changer.

Now, I know what you're thinking - RSS feeds are about as exciting as reading these blog posts. But bear with me because this trick is a game-changer for boosting your content's visibility on platforms like Mastodon. How? By injecting some personality into your social media posts with eye-catching images and hashtags. Trust me, it's worth the effort.

Picture this: I've been automatically publishing my blog posts on Twitter and Mastodon using the dynamic trio of Zapier, Buffer, and my trusty RSS feed. It's been a *mostly* seamless process, but there was one tiny hiccup. The default RSS feed didn't include everything I wanted - specifically, the lack of images and hashtags.

Now, cue the internet search saga. I scoured the web, stumbled upon outdated advice, but ultimately found my beacon of hope - the Hugo documentation on RSS Templates. Surprise, surprise, the documentation turned out to be a goldmine of information. Kudos to Hugo for being on point!

Now, let's cut to the chase. Here's the RSS template I ended up with:

```go
{{- $pctx := . -}}
{{- if .IsHome -}}{{ $pctx = .Site }}{{- end -}}
{{- $pages := slice -}}
{{- if or $.IsHome $.IsSection -}}
{{- $pages = $pctx.RegularPages -}}
{{- else -}}
{{- $pages = $pctx.Pages -}}
{{- end -}}
{{- $limit := .Site.Config.Services.RSS.Limit -}}
{{- if ge $limit 1 -}}
{{- $pages = $pages | first $limit -}}
{{- end -}}
{{- printf "<?xml version=\"1.0\" encoding=\"utf-8\" standalone=\"yes\"?>" | safeHTML }}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ if eq  .Title  .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{.}} on {{ end }}{{ .Site.Title }}{{ end }}</title>
    <link>{{ .Permalink }}</link>
    <description>Recent content {{ if ne  .Title  .Site.Title }}{{ with .Title }}in {{.}} {{ end }}{{ end }}on {{ .Site.Title }}</description>
    <generator>Hugo -- gohugo.io</generator>
    <language>{{ site.Language.LanguageCode }}</language>{{ with .Site.Author.email }}
    <managingEditor>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</managingEditor>{{end}}{{ with .Site.Author.email }}
    <webMaster>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</webMaster>{{end}}{{ with .Site.Copyright }}
    <copyright>{{.}}</copyright>{{end}}{{ if not .Date.IsZero }}
    <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
    {{- with .OutputFormats.Get "RSS" -}}
    {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
    {{- end -}}
    {{ range $pages }}
    <item>
      <title>{{ .Title }}</title>
      <link>{{ .Permalink }}</link>
      <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
      {{ with .Site.Author.email }}<author>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</author>{{end}}
      <guid>{{ .Permalink }}</guid>
      <description>{{ .Params.description | html }}</description>
      <image>{{ .Params.image | absURL }}</image>
      <hashtags>{{ range .Params.tags }}#{{replace . " " "_"}} {{ end }}</hashtags>
    </item>
    {{ end }}
  </channel>
</rss>
```
It's a slightly tweaked version of the [Hugo v0.115.0 template](https://github.com/gohugoio/hugo/blob/release-0.115.0/tpl/tplimpl/embedded/templates/_default/rss.xml) (I really ought to update to the latest version of Hugo), but with the cherry on top - the image and hashtags tags. I made sure the image is an absolute URL for that extra oomph (like, so it works) and grabbed hashtags from the tags parameter, neatly formatted with # and spaces replaced by underscores so it would actually work as a hashtag. Neat!

This is my first rodeo with this setup, and I'm eager to see the results. Local testing seems positive anyway, so it should just be a matter of hooking the new fields into Zapier to format the post for Buffer.

```xml
<rss version="2.0">
   <channel>
   <title>Development and Dinosaurs</title>
   <link>http://localhost:1313/</link>
   <description>Recent content on Development and Dinosaurs</description>
   <generator>Hugo -- gohugo.io</generator>
   <language>en-gb</language>
   <lastBuildDate>Fri, 05 Jan 2024 00:00:00 +0000</lastBuildDate>
   <atom:link href="http://localhost:1313/index.xml" rel="self" type="application/rss+xml"/>
   <item>
      <title>
      Revamping Your RSS Feed - A Social Media Boost for Your Hugo Blog
      </title>
      <link>revamping-your-rss-feed-a-social-media-boost-for-your-hugo-blog/</link>
      <pubDate>Sat, 06 Jan 2024 00:00:00 +0000</pubDate>
      <guid>revamping-your-rss-feed-a-social-media-boost-for-your-hugo-blog/</guid>
      <description>
      Hey peeps! Ever felt like your blog posts were missing a little extra flair when shared on 
      social media? Well, you're not alone. In my quest to supercharge my Twitter and Mastodon game, 
      I stumbled upon a nifty solution involving customizing my RSS template for Hugo. And let me 
      tell you, it's been a game-changer.
      </description>
      <image>
      http://localhost:1313/images/posts/revamping-your-rss-feed-a-social-media-boost-for-your-hugo-blog-banner.png
      </image>
      <hashtags>#hugo #blogumentation </hashtags>
   </item>
   </channel>
</rss>
```

Fingers crossed for a boost in social media engagement! 🚀 I mean I guess, I mostly just want to see my #gamedev posts in the #gamedev feed on Mastodon. 

Until next time. 
