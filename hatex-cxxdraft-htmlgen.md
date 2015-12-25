# The HaTeX parser and how it's used by cxxdraft-htmlgen

HaTeX is a Haskell library for creating and modifying LaTeX documents.
(url)

Its parser component tries to reproduce the document structure of a LaTeX document as a tree of
* commands (`TeXComm`, `TeXCommS`)
* environments (`TeXEnv`)
* comments (`TeXComment`)
* mathematical expressions (`TeXMath`)
* "raw" text passages (`TeXRaw`)

The tree is built as a recursive, unbalanced structure of `TeXSeq` pairs, for example:
```
TeXSeq (TeXCommS "mycommand") (TeXSeq (TeXRaw Text.pack "some text") (TeXComment Text.pack "some comment"))
```
While it is possible to nest `TeXSeq` arbitrarily,
it appears that the way the HaTeX parser is written guarantees that the `TeXSeq` are always nested to the right.
```
TeXSeq a (TeXSeq b (TeXSeq c x) )
```
where `a`, `b` and `c` are not `TeXSeq`, but can contain `TeXSeq`.

The HaTeX parser does not expand any macros.
It provides a bijective transformation between the original LaTeX source code and the tree structure, such that
> If `t :: Text` is a syntactically valid LaTeX block, then:
> ```
> fmap render (parseLaTeX t) == Right t
> ```
> -- [Text.LaTeX.Base.Parser documentation](http://hackage.haskell.org/package/HaTeX-3.15.0.0/docs/Text-LaTeX-Base-Parser.html)
That is, no information is lost, not even white spaces.

## White spaces between commands and arguments

The guarantee of HaTeX's parser causes a problem with macro arguments:
```
\mycommand
{foo}
```
Because of the newline between `\mycommand` and its argument, HaTeX produces:
```
TeXSeq (TeXCommS "mycommand") (TeXBraces "foo")
```
instead of
```
TeXSeq (TeXComm "mycommand" [FixArg (TeXRaw "foo")])
```

Similarly, if there is a space character `' '` between the command and its argument.

cxxdraft-htmlgen fixes this issue by removing certain whitespaces between commands and arguments
in a text preprocessing step performed by the `newlineCurlies` function.
Since `newlineCurlies` implements a context-insensitive search-and-replace operation,
it can (in theory) mess with verbatim expressions.

## Multiple arguments

-> what does `moreArgs` do?

```
\term{}{unscoped enumeration}
```
[dcl.enum]p2
is interpreted as
```
TeXSeq (TeXComm "term" []) (TeXBraces (TeXRaw "unscoped enumeration"))
```
but after `moreArgs` (erroneously) as
```
TeXComm "term" [FixArg (TeXRaw "unscoped enumeration")]
```

Similarly in `statements.tex` for several occurrences of `\grammarterm{}{..}`

Additionally, there's `\discretionary{}{}{}` which controls line breaks
(the implementation of the custom `\brk` macro), as well as
`\list{}{}` (creates a list without markers, used to indent text in the `indented` environment)
and `\list{}{..}` (used in the environment `bnf` for indentation).


What does `\\hspace\*` -> \\hspace` do? A: it's a simplification (treat all of them equally)
Why is hspace in both `kill` and `simpleMacros` in Render.hs?
What does `concatRaws` do? A: In `replaceArgsInString`, it concatenates the TeXRaws which all contain only one character. In `show (concatRaws name')`, it looks suspicous to me. In `parseIndex`, ???

What's the point of `not ("\"" \`Text.isSuffixOf\` x)` in `texStripInfix`?


## Verbatim macros and environments

Because HaTeX's parser does not expand macros, it also doesn't change its "state" during parsing.
Therefore, it cannot parse verbatim environments and macros correctly, such as:
```
\begin{codeblock}
int main() {}
\end{codeblock}

\verb|a\b|
```
Worse, it even hiccups on `regex.tex`,
because it attempts to parse some verbatim strings which produces parsing errors.

cxxdraft-htmlgen's solution consists of two distinct patches:
1. The `\verb` command is resolved via text preprocessing, before the HaTeX parser is run.
2. The verbatim environments are fixed by running the parser twice
  and adding parsing protection (escaping) to the code between the two passes.

### The `\verb` command

For the `\verb` command, a dedicated preprocessing & parsing routine has been written: `translateVerb`.
It seraches for occurrences of the string `"\verb"`,
then proceeds by implementing the special macro expansion of this command in Haskell.
`translateVerb` consumes the next character after `"\verb"` and searches for its next occurrence.
The string between those two occurrences is protected and wrapped inside argument braces for the `\verb` command:
`\verb|a\b|` becomes `\verb{a\\b}`.

### Verbatim environments

Verbatim environments: `codeblock`, `itemdecl`

It has been observed that the verbatim environments in the draft currentely do not cause parsing errors.
Therefore, they're fixed after a first pass of the HaTeX parser.
This first pass is used to identify the verbatim environments.
The LaTeX within those environments then is reversed to plain text (LaTeX source) and protected ("escaped").
Because verbatim environments can contain LaTeX commands,
some additional work has to be done to only protect those parts that are to be interpreted as verbatim.
Essentially, this emulates the work of `\lstlisting`.
(If HaTeX were able to expand `\lstlisting`, this step would be unnecessary.)


## The `\@` command

This command is used to modify the interpretation of a `.` in LaTeX.
Consider:
```
The author is B. Stroustrup       # dot does not denote end of sentence
Classes are not available in C\@. # dot denotes end of sentence
```
This command is used for spacing in LaTeX.
(Note: I do not know if `\@` is actually a LaTeX command,
or just triggers some magic in another way.)

There is a similar spacing issue when the dot shall not denote the end of a sentence:
```
class templates etc.\ are         # dot does not denote end of sentence
class templates etc.\@ are        # dot does not denote end of sentence
```
Both versions should be effectively equivalent.

Since the `'@'` character is not considered as text, HaTeX does not parse `"\@"` as a single command,
but rather as an empty unnamed command `'\'` followed by a raw `'@'` character interpreted as text:
```
TeXSeq (TeXCommS "") (TeXRaw Text.pack "@")
```

It is possible to fix this issue via a tree transformation.
Unfortunately, this transformation needs to look both ahead and back.
The current implementation of `reparseAtCommand` assumes the `TeXSeq` are appended to the right,
and therefore uses a simplistic pattern matching to find potential occurrences of `\@` in relation to a dot.

`reparseAtCommand` is currently run in both passes of the `doParseLaTeX` function.
This is inappropriate, since it can be applied to verbatim text and is not correctly re-rendered
(due to the lack of a dedicated renderer).


## Spacing commands

Similarly to the `\@` command, spacing commands such as `\>` are not parsed correctly by the HaTeX parser.
They produce `TeXSeq (TeXCommS "") (TeXRaw Text.pack ">")`.
The specific spacing command `\>` aka `\medmuskip` is only used in
* `bnftab`-style environments (primary usage)
* table emulation in [class.gslice] via `tabbing` environments
* indentation for listing-emulation in footnotes via `tabbing` environments (see [dyp's harmonize branch](https://github.com/dyp-cpp/cpp-draft/commit/a1811f321603e1808f8f917b5718f3bb83cc1fb9))

cxxdraft-htmlgen currently only replaces `\>` and only inside `bnftab` and `tabbing` environments,
in the tree transformation between the two parser passes (see the `reparseTabs` function).
This replacement is done by re-rendering already an parsed HaTeX tree to text,
then text-replace `"\\>"` with `"\t"`.


## Math mode

The old TeX-style display-math macros `$$ x = 1 $$` are used, though rarely,
in the LaTeX sources of the C++ Standard draft.
HaTeX does not parse those correctly; it interprets the double dollar as an empty display-math and produces
```
TeXSeq (TeXMath Dollar TeXEmpty) (TeXSeq (TeXRaw " x = 1 ") (TeXMath Dollar TeXEmpty))
```
See also [TeX Stackexchange: Why is `\[ … \]` preferable to `$$ … $$`?](http://tex.stackexchange.com/questions/503/why-is-preferable-to).

cxxdraft-htmlgen fixes this by text preprocessing, replacing `"$$"` with `"$"`.
Unfortunately, this turns display-math expressions into inline-math expressions.
