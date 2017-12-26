---
title: A Book
author:
  - An Author
  - Another Author


---


# Chapter Title

This is the introduction paragraph of a chapter.

But that's not the point.  The point is that I need a system of publishing that supports code listings and figures.  I also need that system to produce good quality books in PDF, HTML and EPUB output.  Moreover, as much as I love LaTeX it's not a suitable markup for this job.  In this case I need something that supports a subset of the features found in all three publishing options.  As such [pandoc](http://pandoc.org/) and it's extended Markdown are a good substitute.  They need a little pushing around in order to work as I'd like them.  The rest of this document explains my publishing setup.  You have to assume that the source documents are written in Markdown and named `00_chapter1.md`, `01_chapter2.md` and so on.

It's mostly about finding a good set of Markdown options that suit only those publishing features that I'm using right now.

## Section Title

This is a section within a chapter and we can have a code listing within it

```haskell
qsort [a] -> [a]
qsort []   = []
qsort x:xs = (filter (< x) xs) ++ x ++ (filter (> x) xs)
```

but we need to use the `fignos` extension to be able to reference a figure like figure {@fig:id}.

![This is an image caption](images/image.png){#fig:id}

As usual, equations can be typeset

$$
x = \sum_{i=0}^n i
$$

## More on Diagrams

I generate my diagrams using [Inkscape](http://www.inkscape.org) which produces SVG files.  I then use a `Makefile` to compile the SVGs into whatever I need.  Normally I need PDF files for use with `pdflatex` but in this case I need to produce both PDF output for LaTeX and PNG output for the epub and HTML versions.  A suitable `Makefile` would contain the following which assumes your SVG images are stored in the `images/` subdirectory:

```makefile
SVGS = $(wildcard images/*.svg)
PDFS = $(patsubst %.svg,%.pdf,${SVGS})
PNGS = $(patsubst %.svg,%.png,${SVGS})

INKSCAPE = inkscape

images: ${PDFS} ${PNGS}

images/%.pdf: images/%.svg
	@${INKSCAPE} --without-gui --export-pdf=$@ $<

images/%.png: images/%.svg
	@${INKSCAPE} --without-gui --export-png=$@ $<
```

As you can see, we use [Inkscape](http://www.inkscape.org) in headless mode to produce the PDF and PNG output.  This leaves us with a final wrinkle.

In LaTeX you can

```latex
\includegraphics{images/foo}
```

and leave off the file extension.  The compiler will then assume you want to use PDF images if the compiler is `pdflatex`, users of ye olde `latex` compiler will find that it grabs EPS files by default (note: I stay away from EPS due to the way it doesn't really support transparency very well).  So we have to process the intermediate .tex file output by `pandoc` in order to choose vector based PDF files over the raster PNG format.  We use an _in place_ `sed` to replace any PNG included graphics with the same file name without the specified extension.

```bash
sed -i '.png}' '.pdf}'
```

### Floating Figures

About 80% of the work done above is to replicate a small subset of LaTeX's functionality.  One thing I can't replace is putting code listings in floating figures.  If you know a way around this, please clone this repo and add it.
