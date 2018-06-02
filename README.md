
# Solo

Solo is a simple script that allows you to compile tex files that don't have
a preamble.

## But why?

I usually write my documents in a very non-linear fashion: I have an idea for
a paragraph and I start writing it right away, without even knowing where it
will fit in the greater scheme of my project. Sometimes I like to create new
files for these stray excerpts. It just doesn't feel right to keep them with
the rest of the text for ages until I figure out where they belong. So
I create a tex file just for them, and I don't even bother writing a preamble
-- when all is said and done, those lines will be moved to a different file
anyway. The thing is, sometimes I want to compile one of these files, but then
I'd have to add a whole preamble that is often longer than the text itself.
I might even have many of these little files, and it makes no sense to have 10
files with the same preamble and a different paragraph each. There are a few
LaTeX packages that deal with this, such as the
[subfiles](https://ctan.org/pkg/subfiles) and
[standalone](https://ctan.org/pkg/standalone) packages, but they still require
you to write a small preamble for each file, and that can be enough to disrupt
my workflow. I want to fire up a new Vim buffer and just start typing. That's
the whole point of writing a random paragraph by itself: I want to jot those
ideas down as soon as possible. That's where Solo comes in: if I ever want to
compile one of those files, I can just run `solo file.tex` and get a shiny pdf
to check how things are looking.

Another scenario I encounter frequently is needing to write a casual note or
something like that. Say an archaic online form requires me to attach some
personal details in a pdf file. Now what? I don't want to take the time to
write those unbearable 13 characters, d-o-c-u-m-e-n-t-c-l-a-s-s, just to get
a pdf with my address. And Microsoft Word is obviously not an honorable
option. So what I've been doing so far is to write those small notes as
Markdown files and use Pandoc to convert them to pdf (by first converting them
to LaTeX using templates that are a pain to edit and manage). But not anymore!
Now I can just write my quick notes in LaTeX and use Solo to get a pdf.

## Installation

Just download the `solo` script, make sure it's executable and add it to your
path.

Here's a one-liner, in case you're as lazy as I am (you shouldn't use it if you
don't know what it does, though):

    curl -fsSL https://goo.gl/LkwDwa > /usr/local/bin/solo && chmod +x /usr/local/bin/solo

## Usage

The easiest way to use Solo is to just write a small tex file and run `solo
file.tex`. That will compile your file using this really simple default
template:

```tex
\documentclass{article}
\usepackage{iftex}
\ifPDFTeX
  \usepackage[T1]{fontenc}
  \usepackage[utf8]{inputenc}
  \usepackage{lmodern}
\else
  \usepackage{fontspec}
  \defaultfontfeatures{Ligatures=TeX}
\fi
\usepackage{microtype}
\begin{document}
    % Your file's cotents will go here
\end{document}
```

If you want to add something else to the preamble, you can add it to the top
of your file and then call the `\begin{document}` and `\end{document}`
commands yourself (that kind of defeats the purpose, though, so don't do it).

So an example file might look like this:

```tex
\section{Test}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Ut purus elit,
vestibulum ut, placerat ac, adipiscing vitae, felis. Curabitur dictum gravida
mauris. Nam arcu libero, nonummy eget, consectetuer id, vulputate a, magna.
Donec vehicula augue eu neque. Pellentesque habitant morbi tristique senectus
et netus et malesuada fames ac turpis egestas. Mauris ut leo. Cras viverra
metus rhoncus sem. Nulla et lectus vestibulum urna fringilla ultrices.
Phasellus eu tellus sit amet tortor gravida placerat. Integer sapien est,
iaculis in, pretium quis, viverra ac, nunc. Praesent eget sem vel leo ultrices
bibendum. Aenean faucibus. Morbi dolor nulla, malesuada eu, pulvinar at,
mollis ac, nulla. Curabitur auctor semper nulla. Donec varius orci eget risus.
Duis nibh mi, congue eu, accumsan eleifend, sagittis quis, diam. Duis eget
orci sit amet orci dignissim rutrum.
```

Or like this:

```tex
\usepackage{lipsum}
\begin{document}
    \section{Test}
    \lipsum[1]
\end{document}
```

### Adding packages

You can also specify additional packages with the `-p` flag, so that last
example could be reduced to this:

```tex
\section{Test}
\lipsum[1]
```

As long as Solo were called like this:

    solo -p lipsum file.tex

Multiple packages can be added by using the `-p` flag multiple times:

    solo -p lipsum -p enumitem file.tex

### Modifying the preamble

The `-p` flag doesn't allow you to add options to the packages, but you can
also modify the preamble by using the `-l` flag, which allows you to add
arbitrary lines:

    solo -l '\usepackage[inline]{enumitem}' file.tex

Note that the `-p` and `-l` flags are meant to be used very sparingly; if you
need to make changes to the preamble, you're probably better off writing
a full tex document with it's own preamble. The point here is to focus on the
text and to compile it only provisionally until it finds a more permanent
home.

### Using templates

You can override the default template by adding a file called `default.tex` to
your file's directory or to the `~/.solo` directory. The contents of that file
will be used before `\begin{document}` is called by Solo. If there's anything
you want to add to the end of the document (like `\printbibliography`), place
it in a file called `default_after.tex` and it will be used right before
`\end{document}`.

Besides the default template, you can also create additional templates at the
`~/.solo` directory. If you want to create a template called "receipt", for
instance, you should add the `receipt.tex` and (optionally)
`receipt_after.tex` files to that path. After creating your custom template,
you can use it with the `-t` flag:

    solo -t receipt file.tex

You can also specify the template with a comment in the tex file you want to
compile:

    % solo: receipt

### Choosing the TeX engine

By default, Solo uses `pdflatex` to compile your document. You can change that
by using the `-e` flag:

    solo -e lualatex file.tex

If you always want to use a specific engine with one of your templates, you
can specify it in a comment *in the template file* (not in the file that
contains the document's text):

    % solo: lualatex

### Processing bibliographies

If you also want Solo to process your file's citations and bibliographies, you
can specify the program to be used with the `-b` flag:

    solo -b biber file.tex

After compiling your file, Solo will run the specified program and then it'll
run the TeX engine two more times.

If you always want to process bibliographies when using a template, you can
specify it in a comment *in the template file*:

    % solo: biber

