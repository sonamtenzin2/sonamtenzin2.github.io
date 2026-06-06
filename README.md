# sonamtenzin2.github.io

Personal blog and portfolio site built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, hosted on GitHub Pages.

## Development

```bash
cd website-personal
hugo new content content/posts/post-name.md     # Create new post
hugo server          # Start dev server with live reload
hugo server --disableFastRender  # Dev server with full rebuilds
```

## Build

```bash
hugo --gc --minify --source ./website-personal
```

## Deployment

Pushing to `main` triggers a [GitHub Actions workflow](.github/workflows/hugo.yaml) that builds and deploys to GitHub Pages.
