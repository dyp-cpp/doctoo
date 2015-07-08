This is a short overview of tools that can convert the LaTeX sources to XML or some other format.

# HaTeX

Haskell library which parses/tokenizes LaTeX code into a custom (Haskell) tree data structure
parses the LaTeX source assuming that the parsing won't be affected by macro expansion

problems:
- the rudimentary way it "parses" LaTeX causes issues with verbatim environments and probably with document class files / style files as well
- doesn't parse whitespace between commands and argument-braces correctly
- both problems are related to the fact that it guarantees the invariant `fmap render (parseLaTeX t) == Right t`

used by: [cxxdraft-htmlgen](https://github.com/Eelis/cxxdraft-htmlgen)


# LXir

embeds tags into DVI and then parses the DVI
written in C and LaTeX (embedding macros)

notes:
seems not to be able to cope with memoir, as it requires custom document classes for tagging. Replacing memoir by book worked fine.
The results seem nice, it even correctly convertes the special characters such as (R).

problems:
- verbatim environment strips all formatting
- per default:
  - produces html, not xml
  - too much formatting (awful lot of cascaded divs and spans)
- documentation in French only
- support for lstlisting environments is poor: the output is not treated correctly (formatting is lost in output **processing**). However, you can hack it by inserting visible white spaces, newline tags (via lstlisting's line number styles) and process the result via a custom XSLT. NOTE: verbatim environments are handled correctly (spaces and newlines are replaced).. maybe I can make that work with listing envs, too?
- text style (e.g. italic) & font dropped in some transformation (recoverable, but will use special semantic markup instead)
- \paragraph and \subparagraph don't produce sectionHeaders (workaround possible)

used by: [cppdraft2xml](https://github.com/dyp-cpp/cpp-draft/tree/xml)


# latexml

written in python and uses python scripts for extensions

problems:
- strict DocType checks, doesn't (easily) allow injecting own tags into their document structure
-> not sure if you can do this at all, maybe the document structure had to be rewritten entirely (=> what about the verbatim/listing environment?)


# tralics

written in C++ and uses (La)TeX for extensions
parses (La)TeX code directly

notes:
- seems to require a documentclass, otherwise it behaves strangely

problems:
- seems not to be able to parse native tex style files (efficiently)
- packages typically have to be rewritten (and much simplified)
- listings package doesn't support lstnewenvironment, and doesn't seem to support any escaping. The Standard uses automatic "escaping" in comments; i.e. you can write (La)TeX commands in comments w/o escaping
-> it seems you have to manually enhance the included minimal listings implementation to support the features used in the Standard



# Hermes

embeds tags into DVI and then parses the DVI
written in C and LaTeX (embedding macros)

notes:
uses an arbitrary LaTeX engine to do the TeX -> DVI conversion

problems:
- not extensible (well you can hack the LaTeX part, for example)
- seems to have some encoding problems (special chars like (c))
- support for verbatim/listings seems to be poor (all formatting lost)


# pandoc

notes: you CAN write a custom "writer" in Lua; not sure if you can embed something into the LaTeX sources, though.
Reads special characters correctly.

problems:
- the AST output suggests that the whitespaces are merged and there's no difference between \n and ' '. Verbatim environments won't work therefore, it seems.
- When using -R -p, the codeblocks (and some other things like \pnum) are not translated. It seems they're "ignored" otherwise (text just printed). Unfortunately, this means that you'll either get a modified output (spaces and newlines mangled) or you'll get a completely untranslated output, where TeX macros inside comments are not translated either.
