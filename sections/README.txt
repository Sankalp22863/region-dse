Editing this blog
=================

Every section of the post lives in its own .txt file here. Edit any of them and
reload the page — no build step, nothing to recompile.

  manifest.txt   which section files render, and in what order
  00-meta.txt    title, subtitle, tags, byline, date, header links
  NN-*.txt       one section of the post each

MARKUP (all optional — a plain paragraph on its own line just works)

  ## Heading                 a section heading
  ### Subheading             a smaller heading
  plain text                 a paragraph
  - item                     a bulleted list (consecutive lines group)
  1. item                    a numbered list
  > text                     a highlighted callout block
  ---                        a thin divider

INLINE

  **bold**      *italic*      `code`      [label](https://url)
  10^4          renders as a superscript exponent
  --            renders as an em dash

BLOCKS

  [[TLDR]]                   the grey TL;DR box, collapsible, starts OPEN
  [[TLDR:closed]]            same box, but starts collapsed
  [[FIG:name]]               drops in a figure. Valid names:
                               feasibility  clustering  algorithm  budget
                               latency      walltime    scaling    stentor
  [[STATS]]                   a stat grid; one per line, until a blank line:
                                value | label | note
  [[REF]] authors | title | venue | url

NOTES

  * The page fetches these files at runtime, so it must be served over HTTP:
        python3 -m http.server 8000
    and opened at http://127.0.0.1:8000/  — opening index.html as a file://
    URL will not work (the browser blocks local fetches).
  * Figure DATA lives in index.html, in the DATA object near the top of the
    script. Prose lives here; numbers live there.
  * index.html and main.html are the same page; both read these files.
  * URL flags, handy for screenshots and printing:
        ?static=1   freeze all animations and count-ups at their final values
        ?tables=1   open every figure's "Show data" table
    Animations are also frozen automatically for readers whose system asks for
    reduced motion.
