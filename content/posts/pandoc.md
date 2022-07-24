---
title: "What I learned: Pandoc"
date: 2019-06-30T13:52:11+02:00
tags: ["whatilearned"]
draft: false
---
I love using LaTeX for creating documents and slides. I chose it over
alternatives like Word (because IMO Word behaves inconsistenly and I take much
longer to fiddle with the options to get a nice document) or [reveal.js](https://github.com/hakimel/reveal.js)
(because of it's complex file structure and I don't get along with node).

The only problem with LaTex is that it's pretty outdated and the language
itself is filled with redundancy. My friend Ben Hilburn wrote a [blog post](https://bhilburn.org/latex-needs-to-be-reborn/)
about this as well. I recently began to work around the pain of plain LaTeX by
using [Pandoc](https://github.com/jgm/pandoc). It is a very handy open source
tool that provides functionality to convert documents between different formats. In
particular, it lets me write my document in **another** language, before
converting it internally into LaTeX and then producing a PDF.

# How it works

I chose to write my documents in [Markdown](https://daringfireball.net/projects/markdown/syntax)
instead of LaTeX. This works well both for print-out documents and slides. For
instance, the highest hierachy of sections in a Markdown document will become
the sections of the LaTeX slides.  Subsections will become frame titles and so
on. To give an examaple, this:

```tex
\begin{frame}{Super Important List}
\begin{itemize}
\tightlist
\item
  \texttt{master} branch is default branch in git
\item
  \texttt{feature} branch is where new features get implemented
\item
  Use a pull request to get changes from a feauture branch onto master
\end{itemize}
\end{frame}
```

becomes this in Markdown:

```markdown
## Super Important List

- `master` branch is default branch in git
- `feauture` branch is where new features get implemented
- Use a pull request got get changes from a feature branch onto master
```

You see a lot of redundancy is gone with Markdown, which directly results in
faster document creation. You can even use LaTeX code in Markdown, which will
be untouched at the conversion from Markdown into LaTeX and then be processed
next to the generated LaTeX code.

Print documents etc. work all in the same manner. Here is an example of how my
Markdown file for a letter looks like:

```markdown
---
fromname: Your Name
fromstreet: YourSreet 4
fromzip: 12345 München
fromemail: my@mail.com
recipient: |
    Title

    Auguststr. 3

    11234 Berlin
subject: "Wichtige Information"
#salutation: "Sehr geehrte Frau Maier,"
#signature: 1  # place sig.pdf in same folder
place: "München"
#date: "10. Mai 2020"
...

Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod
tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At
vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd
gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum
dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor
invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero
eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no
sea takimata sanctus est Lorem ipsum dolor sit amet.
```

This short Markdown code produces a beautiful PDF. Note that it has a YAML
header, which gets parsed by Pandoc as well, to fill the fields of the letter
with the respective information.

<img src="/images/letter.png" align="middle" style="display:block;margin-left:auto;margin-right:auto;width:80%;box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);" />

To put it another way, Markdown is a more efficient language to transport the
same information as LaTeX in the majority of cases. So why not use it?

# Setup

I wrote a little `Makefile` for Pandoc projects, which looks like this:

```
FILENAME = letter

${FILENAME}.pdf: ${FILENAME}.md
    pandoc -t latex -s --template=template.letter letter.md -o letter.pdf

clean:
    rm letter.pdf
```

There you can see the `pandoc` command and it's araguments. I'm using a
template file in that case, that just tells Pandoc how to create the
intermediate TeX file.

The source for my [slides](https://github.com/sbmueller/pandoc_beamer) and
[letter](https://github.com/sbmueller/pandoc_letter) templates are available on
GitHub, if you want to dive deeper into the topic.
