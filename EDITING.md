# Editing this site

Everything you'll want to change day-to-day lives in one of five places. You never
need to touch HTML for news, papers, or the CV.

## Run it locally

Homebrew Ruby is required (the macOS system Ruby 2.6 is too old for bundler):

```
PATH="/opt/homebrew/opt/ruby/bin:$PATH" RUBYOPT="-r/tmp/untaint_shim.rb" bundle exec jekyll serve --port 4000
```

Then open http://127.0.0.1:4000. Edits rebuild automatically; `_config.yml` is the one
file that needs a server restart.

If `/tmp/untaint_shim.rb` is gone (it clears on reboot), recreate it:

```
printf 'class Object\n  def untaint; self; end unless method_defined?(:untaint)\nend\n' > /tmp/untaint_shim.rb
```

It exists because Liquid 4.0.3 calls `String#untaint`, removed in Ruby 3.2+.

---

## News → `_data/news.yml`

One entry per item, newest first. `recent: true` shows it on the page; leave it off and
the item drops into the collapsed "Older news" section.

```yaml
- date: Sept 2026
  recent: true
  html: >-
    Our work on <a target="_blank" href="https://...">mechanistic finetuning</a> was
    accepted as an <b>oral presentation</b> at ECCV 2026!
```

The `>-` lets you write plain HTML across multiple lines without escaping quotes.
Indent continuation lines. Both the current site and the drafts read this same file.

## Papers → `items/_posts/`

One Markdown file per paper, named `YYYY-MM-DD-shortname.md`. The date controls which
year heading it appears under and its order within that year. Front matter:

```yaml
---
layout: post
title:  "Paper Title"
date:   2026-06-01 0:0:0 +00:00
categories: research          # required — without this it won't show up
image: /assets/images/index/foo.png
gif:   /assets/images/index/gifs/foo.gif   # optional; shown instead of image
authors: Some One, <b>Abrar Anwar</b>, Someone Else
venue: RSS 2026 <font color="#ff0000">(Oral Presentation)</font>
arxiv: https://arxiv.org/abs/...
website: https://...
code: https://github.com/...
---

One or two sentences of description. This is the grey text under the authors.
```

Every link field is optional and each one that's present renders a button: `paper`,
`arxiv`, `tech_report`, `video`, `code`, `poster`, `slides`, `website`, `blog`,
`twitter`. Bold your own name with `<b>Abrar Anwar</b>`.

## Bio → `_includes/bio.html`

The intro paragraphs. Plain HTML, edit directly. This is the only copy — `index.html`
pulls it in.

## Name, photo, email, social icons, nav → `index.html`

All in the `<aside class="side">` block near the bottom of the file. `_includes/header.html`
is the old top nav, still used by `books.html`.

---

## Making GIFs for papers

Requires `ffmpeg`, `gifsicle`, and `yt-dlp` (all installed via Homebrew).

```
ffmpeg -ss START -t DURATION -i source.mp4 \
  -vf "fps=10,scale=300:-2:flags=lanczos,palettegen=max_colors=256:stats_mode=full" \
  -frames:v 1 pal.png
ffmpeg -ss START -t DURATION -i source.mp4 -i pal.png \
  -lavfi "fps=10,scale=300:-2:flags=lanczos[x];[x][1:v]paletteuse=dither=sierra2_4a" \
  -loop 0 out.gif
gifsicle -O3 out.gif -o final.gif
```

Notes from making the current set:
- Aim for 4–6 seconds at 300px wide and 10fps. That lands around 0.8–1.4 MB.
- Do **not** use `gifsicle --lossy` above ~30 or drop below 128 colors — it smears
  moving footage badly.
- For a locked-off camera add `stats_mode=diff` to palettegen and `diff_mode=rectangle`
  to paletteuse; for a moving camera use `stats_mode=full` and omit `diff_mode`.
- To pull source video: `yt-dlp -f "bv*[height<=720]" -o out.mp4 <url>`
- To find a good segment, make a contact sheet first rather than guessing timestamps:
  `ffmpeg -i src.mp4 -vf "fps=1/4,scale=300:-1,tile=6x5" -frames:v 1 sheet.png`

## Drafts

`index.html` is the redesign (formerly `drafts/four.html`). The `drafts/` folder still
holds the other candidates and the GIF/experience preview pages; it's listed under
`exclude:` in `_config.yml` so none of it is published. Delete the folder whenever.
