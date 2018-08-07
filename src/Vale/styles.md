---
layout: section
slug: styles
weight: 3

title: Styles
description: |
  An introduction to Vale, a syntax-aware linter for prose built with speed and
  extensibility in mind.

sections: [
    { title: "Overview" },
    { title: "Extension Points" },
    { title: "Existence" },
    { title: "Substitution" },
    { title: "Occurrence" },
    { title: "Repetition" },
    { title: "Consistency" },
    { title: "Conditional" },
    { title: "Capitalization" },
    { title: "Readability" },
    { title: "Spelling" }
]

code: true
js: [
  table.js
]
color: green
---

{{% article Overview %}}

Vale has a powerful extension system that doesn't require knowledge of any programming languages. Instead, it exposes its functionality through simple
[YAML](http://yaml.org) files.

The core component of Vale's extension system are collections of writing guidelines called *styles*. These guidelines are expressed through *rules*, which are YAML files enforcing a particular writing construct&mdash;e.g., ensuring a certain readability level, sentence length, or heading style.

Styles are organized in a hierarchical folder structure at a user-specified location (see [Configuration](/vale/config/) for more details). For example,

```
styles/
├── base/
│   ├── ComplexWords.yml
│   ├── SentenceLength.yml
│   ...
├── blog/
│   ├── TechTerms.yml
│   ...
└── docs/
    ├── Branding.yml
    ...
```

where *base*, *blog*, and *docs* are your styles.

{{% /article %}}

{{% article "Extension Points" %}}

{{% alert primary %}}

**NOTE**: Vale uses Go's [`regexp` package](https://golang.org/pkg/regexp/syntax/) to evaluate all patterns in rule definitions. This means that lookarounds and backreferences are not supported.

{{% /alert %}}

The building blocks behind Vale's styles are its rules, which utilize extension points to perform specific tasks.

The basic structure of a rule consists of a small header (shown below) followed by extension-specific arguments.

```yaml
# All rules should define the following header keys:
#
# `extends` indicates the extension point being used (see below for information
# on the possible values).
extends: existence
# `message` is shown to the user when the rule is broken.
#
# Many extension points accept format specifiers (%s), which are replaced by
# extracted values. See the exention-specific sections below for more details.
message: "Consider removing '%s'"
# `level` assigns the rule's severity.
#
# The accepted values are suggestion, warning, and error.
level: warning
# `scope` specifies where this rule should apply -- e.g., headings, sentences, etc.
#
# See the Markup section for more information on scoping.
scope: heading
# `code` determines whether or not the content of code spans -- e.g., `foo` for
# Markdown -- is ignored.
code: false
# `link` gives the source for this rule.
link: 'https://valelint.github.io/docs/styles/#creating-a-style'
```

{{% /article %}}

{{% article existence %}}

{{< api existence 1 1 2 >}}

The most general extension point is `existence`. As its name implies, it looks
for the "existence" of particular tokens.

These tokens can be anything from simple phrases (as in the above example) to complex regular expressions&mdash;e.g., [the number of spaces between sentences](https://github.com/errata-ai/vale-boilerplate/blob/master/src/18F/Spacing.yml) and [the position of punctuation after quotes](https://github.com/errata-ai/vale-boilerplate/blob/master/src/18F/Quotes.yml).

You may define the tokens as elements of lists named either `tokens`
(shown above) or `raw`. The former converts its elements into a word-bounded,
non-capturing group. For instance,

```yaml
tokens:
  - appears to be
  - arguably
```

becomes `\b(?:appears to be|arguably)\b`.

`raw`, on the other hand, simply concatenates its elements&mdash;so, something
like

```yaml
raw:
  - '(?:foo)\sbar'
  - '(baz)'</code></pre>
```

becomes `(?:foo)\sbar(baz)`.

{{% /article %}}

{{% article substitution %}}

{{< api substitution 2 3 4 >}}

`substitution` associates a string with a preferred form. If we want to suggest
the use of "plenty" instead of "abundance," for example, we'd write:

```yaml
swap:
  abundance: plenty
```

The keys may be regular expressions, but they can't include nested capture groups:

```yaml
swap:
  '(?:give|gave) rise to': lead to # this is okay
  '(give|gave) rise to': lead to # this is bad!
```

Like `existence`, `substitution` accepts the keys `ignorecase` and `nonword`.

`substitution` can have one or two `%s` format specifiers in its message. This allows us to do either of the following:

```yaml
message: "Consider using '%s' instead of '%s'"
# or
message: "Consider using '%s'"
```

{{% /article %}}

{{% article occurrence %}}

{{< api occurrence 3 5 6 >}}

`occurrence` limits the number of times a particular token can appear in a given scope. In the example above, we're limiting the number of words per sentence.

This is the only extension point that doesn't accept a format specifier in its message.

{{% /article %}}

{{% article repetition %}}

{{< api repetition 4 7 8 >}}

`repetition` looks for repeated occurrences of its tokens. If `ignorecase` is set to `true`, it'll convert all tokens to lower case for comparison purposes.

{{% /article %}}

{{% article consistency %}}

{{< api consistency 5 9 10 >}}

`consistency` will ensure that a key and its value (e.g., "advisor" and "adviser") don't both occur in its scope.

{{% /article %}}

{{% article conditional %}}

{{< api conditional 6 11 12 >}}

`conditional` ensures that the existence of `first` implies the existence of `second`. For example, consider the following text:

> According to Wikipedia, the World Health Organization (WHO) is a specialized agency of the United Nations that is concerned with international public health. We can now use WHO because it has been defined, but we can't use DAFB because people may not know what it represents. We can use DAFB when it's presented as code, though.

Running `vale` on the above text with our example rule yields the following:

```shell
test.md:1:224:vale.UnexpandedAcronyms:'DAFB' has no definition
```

`conditional` also takes an optional `exceptions` list. Any token listed as an exception won't be flagged.

{{% /article %}}

{{% article capitalization %}}

{{< api capitalization 6 13 14 >}}

`capitalization` checks that the text in the specified scope matches the case of `match`. There are a few pre-defined variables that can be passed as matches:

- `$title`: "The Quick Brown Fox Jumps Over the Lazy Dog."
- `$sentence`: "The quick brown fox jumps over the lazy dog."
- `$lower`: "the quick brown fox jumps over the lazy dog."
- `$upper`: "THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG."

Additionally, when using `match: $title`, you can specify a style of either AP or Chicago.

{{% /article %}}

{{% article readability %}}

{{< api readability 6 15 16 >}}

`readability` calculates a readability score according the specified metrics. The supported tests are Gunning-Fog, Coleman-Liau, Flesch-Kincaid, SMOG, and Automated Readability.

If more than one is listed (as seen above), the scores will be averaged. This is also the only extension point that doesn't accept a scope, as readability is always calculated using the entire document.

`grade `is the highest acceptable score. Using the example above, a warning will be issued if `grade` exceeds 8.

{{% /article %}}

{{% article spelling %}}

{{< api spelling 7 17 18 >}}

`spelling` implements spell checking based on Hunspell-compatible dictionaries. By default, Vale includes [en_US-web](https://github.com/errata-ai/en_US-web)—an up-to-date, actively maintained dictionary. However, you may also specify your own via the `dic` and `aff` keys (the fully-qualified paths are required; e.g., `/usr/share/hunspell/en_US.dic`).

`spelling` also accepts an `ignore` file, which consists of one word per line to be ignored during spell checking.

Additionally, you may further customize the spell-checking experience by defining *filters*:

```yaml
extends: spelling
message: "Did you really mean '%s'?"
level: error
# This disables the built-in filters. If you omit this key or set it to false,
# custom filters (see below) are added on top of the built-in ones.
#
# By default, Vale includes filters for acronyms, abbreviations, and numbers.
custom: true
# A "filter" is a regular expression specifying words to ignore during spell
# checking.
filters:
  - '[pP]y.*\b'  # Ignore all words starting with 'py' -- e.g., 'PyYAML'.
ignore: ci/vocab.txt
```

{{% /article %}}
