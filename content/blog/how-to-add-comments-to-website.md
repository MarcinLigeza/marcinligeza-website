---
title: "How I Added a Comment Section to My Hugo Posts"
date: 2026-07-10
comments: true
tags: ["Website", "comments", "Hugo", "Giscus"]
summary: "How to add per-post comments to a Hugo site with Giscus and GitHub Discussions."
---
I wanted readers to be able to ask questions and discuss the posts I publish. I chose [Giscus](https://giscus.app/), a comment system built on GitHub Discussions. It keeps the discussion with the repository, requires no separate database, and lets visitors sign in with their GitHub account.

This post shows the complete setup for a Hugo site using the PaperMod theme. The same approach works with other themes: the only theme-specific part is where the comment partial is included.

## What you need first

Giscus works with a public GitHub repository that has GitHub Discussions enabled. Before changing the Hugo site:

1. In the repository settings, enable **Discussions**.
2. Create a category for website comments, such as **Blog comments**.
3. Install the [Giscus GitHub App](https://github.com/apps/giscus) and grant it access to the repository.
4. Open the [Giscus configuration page](https://giscus.app/), select the repository and category, then choose how pages should map to discussions.

I use the `pathname` mapping. With this option, `/blog/a-post/` and `/blog/another-post/` each get their own discussion. Choose the mapping deliberately: changing a post's URL later will create a different discussion unless the existing discussion is migrated too.

The configuration page generates the repository and category IDs needed by the client script. These IDs are not GitHub secrets; they identify the public repository and discussion category.

## Store the Giscus configuration in Hugo

I keep the generated settings in [hugo.yaml](https://github.com/MarcinLigeza/marcinligeza-website/blob/master/hugo.yaml), rather than duplicating them in every template. Replace the example values with the values from the Giscus configuration page:

```yaml
params:
  giscus:
    repo: "your-account/your-site-repository"
    repoId: "R_kgDO..."
    category: "Blog comments"
    categoryId: "DIC_kwDO..."
    mapping: "pathname"
    strict: false
    reactionsEnabled: true
    emitMetadata: false
    inputPosition: "bottom"
    lang: "en"
    theme: "preferred_color_scheme"
    crossorigin: "anonymous"
```

The choices above keep the comment box below the thread, enable reactions, and follow the visitor's system colour preference. The Giscus configuration page can generate other themes and mappings if those are a better fit for the site.

## Add the Giscus partial

The comment partial lives in [layouts/partials/comments.html](https://github.com/MarcinLigeza/marcinligeza-website/blob/master/layouts/partials/comments.html). Hugo reads the values from `site.Params.giscus` and writes them as `data-*` attributes on the Giscus script:

```go-html-template
{{ with site.Params.giscus }}
<script src="https://giscus.app/client.js"
        data-repo="{{ .repo }}"
        data-repo-id="{{ .repoId }}"
        data-category="{{ .category }}"
        data-category-id="{{ .categoryId }}"
        data-mapping="{{ .mapping }}"
        data-strict="{{ if .strict }}1{{ else }}0{{ end }}"
        data-reactions-enabled="{{ if .reactionsEnabled }}1{{ else }}0{{ end }}"
        data-emit-metadata="{{ if .emitMetadata }}1{{ else }}0{{ end }}"
        data-input-position="{{ .inputPosition }}"
        data-theme="{{ .theme }}"
        data-lang="{{ .lang }}"
        crossorigin="{{ .crossorigin | default "anonymous" }}"
        async>
</script>
{{ end }}
```

Keeping the partial configuration-driven makes it easy to change the category, language, or theme later without editing template markup.

## Render comments only where they belong

The custom single-page layout at [layouts/single.html](https://github.com/MarcinLigeza/marcinligeza-website/blob/master/layouts/single.html) renders the partial only when a page has a `comments` parameter:

```go-html-template
{{ if (.Param "comments") }}
  {{ partial "comments.html" . }}
{{ end }}
```

This is useful for keeping comments off pages such as the home page, projects, and static documentation. Enable them on an individual post with front matter:

```yaml
---
title: "My post"
date: 2026-07-10
comments: true
---
```

This post has `comments: true`, so it is also a live test of the integration. Omitting the property, or setting it to `false`, leaves the Giscus script out of that page entirely.

The site also has an author footer in `single.html`. It is rendered independently of the comment flag, so disabling comments does not remove the rest of the post footer.

## Build and verify

After adding the partial and front matter, run a Hugo build and open a post with `comments: true`. The Giscus discussion should appear below the post. When the first visitor creates a comment, Giscus creates or associates a GitHub Discussion according to the selected mapping.

If the area is empty, check the browser console and verify that:

- the repository is public and GitHub Discussions are enabled;
- the Giscus app has access to that repository;
- `repoId` and `categoryId` exactly match the values generated by Giscus;
- the post's front matter includes `comments: true`; and
- the published URL matches the mapping choice, especially when using `pathname`.

Giscus loads content from GitHub, so visitors who block third-party scripts may not see the comment area. Commenting also requires a GitHub account. For a small technical site, those trade-offs are worth avoiding another hosted service and keeping the conversation next to the source for the post.
