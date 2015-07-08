<dl>
<dt>`\tcode{text}`
<dd>Used for inline code, mostly code fragments or single keywords.</dd>

<dt>`\techterm{text}</dt>`
<dd>Not used in the core language parts.</dd>

<dt>`\defnx{text}{index text}`</dt>
<dd>Indexed definition. Used for inline definitions of terms. The first argument is printed in the text, the second is what appears in the index.</dd>

<dt>`\defn{text}`</dt>
<dd>Simplification of `\defnx`, defined as `\defnx{text}{text}`.</dd>

<dt>`\term{text}`</dt>
<dd>Ambiguous use, over 500 occurences.
<ul>
<li>emphasizing, e.g. in [access]:
<pre><code>It should be noted that it is
\term{access}
to members and base classes that is controlled, not their
\term{visibility}.
</code></pre>
The words "access" and "visibility" are not defined via `\defn` nor `\defnx`, and are used without any formatting at other places in [access].
</li>

<li>marking a phrase as a technical term (that is defined somewhere), e.g. in [access]
<pre><code>A base class
\tcode{B}
of
\tcode{N}
is
\term{accessible}
at
\term{R},
if
\begin{itemize}
</code></pre>
though arguably that should be a `\defn` or an unindexed definition.
</li>

<li>for placeholders (should probably replaced with `\placeholder`), see the example above</li>
<li>marking a phrase as a grammar term (despite `\grammarterm`), e.g. in [access]
<pre><code>\enternote A \tcode{friend} declaration may be the
\term{declaration} in a \grammarterm{template-declaration}
</code></pre>
[basic] arguably contains a misuse as `\term{storage} \term{duration}`; [intro] contains `\term{storage duration}`; [basic.stc] has a `\indextext{storage~duration}` but does not format the phrase.
There are several occurences of `\term{something}~(\ref{section})`.
</li>
<li>formatting of "cv", e.g. in [basic]
<pre><code>is to pointer to \term{cv} \tcode{void}, or to pointer to \term{cv}
</code></pre>
This should probably be replaced by `\cvqual`.
</ul>

<dt>`\grammarterm{text}`</dt>
<dd>Marks a phrase as a term defined in the grammar, e.g. *postfix-expression*. Over 2000 occurences. I did not check them, but I think the name is clear.</dd>

<dt>`\placeholder{text}`</dt>
<dd>Occurs only 10 times (in core language).
Marks some "name" as a placeholder, e.g. in [basic]
```
For each value \placeholder{i} of type \tcode{unsigned char} in the range
0 to 255 inclusive, there exists a value \placeholder{j} of type
```
and in [intro]
```
For each signal handler invocation, evaluations
perfomed by the thread invoking a signal handler can be divided into two
groups \placeholder{A} and \placeholder{B}, such that no evaluations in
\placeholder{B} happen before evaluations in \placeholder{A}, and the
```

<dt>`\Rplus`</dt>
<dd>Not used (in core language).</dd>

<dt>`\state`</dt>
<dd>Not used (in core language).</dd>

<dt>NTBS etc.</dt>
<dd>Commands used to write "NTBS", "NTMBS" etc. in small caps. NTBS stands for Null-Terminated Byte String.</dd>

<dt>`\uniquens`</dt>
<dd>Used to write "unique" as the name for unique namespaces in [declarations].</dd>

<dt>`\doccite{text}`</dt>
<dd>Marks some text as a citation of a document. Mostly used in [intro] for normative references.</dd>

<dt>`\cvqual`</dt>
<dd>Marks some text as a cv-qualification (or no cv-qualification), e.g. `\cvqual{cv1}`. Occurs about 100 times.</dd>

<dt>`\cv`</dt>
<dd>Defined as `\cvqual{cv}`, i.e. prints "cv" formatted as a cv-qualification.</dd>

<dt>`\emph{text}`</dt>
<dd>Emphasises. Only used 3 times.</dd>

<dt>`\numconst{text}`</dt>
<dd>Marks some text as a number (literal). Occurs only in [lex] in the escape sequences table and subsequent references to esacpes with a number. E.g.
```
The escape
\indextext{number!octal}%
\tcode{\textbackslash\numconst{ooo}} consists of the backslasgh followed by one,
two, or three octal digits that are taken to specify the value of the
desired character.
```
</dd>

<dt>`\logop{text}`</dt>
<dd>Used probably to format the name of logical operators in what looks like small caps (but using `\footnotesize`). E.g.
```
The usual arithmetic conversions are performed; the result is the
bitwise \logop{AND} functions of the operands.
```
</dd>

<dt>environment `indented`</dt>
<dd>Not used (in core language).</dd>

<dt>`\nonterminal{text}`</dt>
<dd>Marks some text as a nonterminal (inside BNFs).</dd>

<dt>environment `bnfkeywordtab`</dt>
<dd>Used twice (in [lex.operators]/1 and [over.oper]/1). Defines a nonterminal, where the rules are simple terminals and there are so many of them that you cannot (practically) put them on individual lines. In LaTeX code, the rules are separated by `\>` and the line breaks defined via `\br`
</dd>

<dt>environment `bnftab`</dt>
<dd>Defines a nonterminal; the rules are manually indented. Allows additional indentation within the rule [cpp] and line-breaks within a rule, the following line is then continued with one more level of indentation [lex.ccon].
The non-copied version `ncbnftab` is used sometimes (always?) without any manual indentation, and sometimes (always?) without defining a nonterminal inside the environment.
</dd>

<dt>environment `simplebnf`</dt>
<dd>Only occurs as `ncsimplebnf`. Does not contain manual indenting and does not define a nonterminal inside the environment.</dd>

<dt>environment `bnf`</dt>
<dd>Always contains a `\nontermdef`. Defines the nonterminal using one or more rules (automatic indenting).
One exception is [dcl.fct.def.general]/1, where "function-body:" has been used as the nonterminal which is to be defined, instead of `\nontermdef{function-body}` or `\nonterminal{function-body:}`.

The non-copied version usually doesn't contain a nonterminal definition; the single exception being [over.match.call]/3
</dd>

<dt>`\ICS{parameter number}{function name}`</dt>
<dd>This macro has been introduced by dyp's [harmonize branch](https://github.com/dyp-cpp/cpp-draft/tree/harmonize).

In [overloading], more specifically [over.match.best], there is a comparison between the Implicit Conversion Sequences (ICS) for arguments expressions in overload resolution. As we're dealing with multiple parameters and two functions in this comparison, an ICS is identified by the index of the function parameter and the function itself, for example ICS<i>0</i>(my_function).
It is currently written as `ICS\textit{i}(\tcode{F})` in the LaTeX sources.
To remove the meaningless `\textit`, improve readability and formatting, a macro `\ICS{i}{F}` has been introduced.
A similar issue occurs in [conversions] for e.g. `$\mathit{cv}_{1,0}P_0$`
</dd>
