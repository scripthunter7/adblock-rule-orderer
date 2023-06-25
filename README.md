# Adblock Rule Orderer

A simple tool which helps you to order your adblock rules in your filter lists.

## Concept

Currently, there are many adblock filter lists, and they are not ordered in a
consistent way. This makes it harder to put a rule in the right place.

It would be nice if there is a tool which can help us to order the rules in a
consistent way, based on some rules.

In this project, I'll try to design such a tool, you can find the concept
below.

> **Note**: **Please keep in mind that this is just a _very early concept_, and
> it might be changed in the future.**

### Parsing

First of all, we should parse the adblock rules. This can be done by using the
[AGTree](https://www.npmjs.com/package/@adguard/agtree) library, which can
parse any AdGuard, uBlock Origin, AdBlock Plus, etc. rules.

After parsing the rules, we will get a tree structure, which can be used to
traverse the rules and do some operations on them.

### Ordering

When thinking about ordering rules, we should distinguish the following two
categories:

- rule blocks: group of rules
- rule ordering: ordering rules in a block based on some rules

### Rule blocks

Rule blocks are groups of rules. There are two types of rule blocks:

- Rules between `!#if ...` and `!#endif` directives:
  ```adblock
  ! Rule block #1
  rule1
  !#if ...
  ! Rule block #2
  rule2
  rule3
  !#endif
  ! Rule block #3
  rule4
  ```
  - Possible edge case: keep pre-processor comments with the pre-processor comment
    ```adblock
    ! Comment for "if"
    !#if ...
    rule1
    !#endif
    ```
- Rules separated by empty lines

  ```adblock
  ! Rule block #1
  rule1
  rule2

  ! Rule block #2 (because there is an empty line before this line)
  rule3
  rule4
  ```

  - Allow specifying the regexp pattern of the "separator" lines in the
    orderer config file?

### Rule ordering

Ordering rules in a block based on some aspects.

#### Category

First of all, we should consider the category of the rule:

There are 3 categories of rules:

- Comments: `! Comment`, `!#if (adguard)`, `!#endif`, etc
- Cosmetic: `##.ad`, `#%#//scriptlet(...)`, `$$script[tag-content="..."]`, etc
- Network: `/ads.js`, `||example.com^`, etc

And there are 2 more categories of rules (regarding to the parser):

- Invalid rules: rules that cannot be parsed
- Empty lines: empty lines between rules

Rules within the same category also have "types":

- Comment rules have the following types:
  - Pre-processor comments: `!#if (adguard)`, `!#endif`, etc
  - Adblock agent comments: `[Adblock Plus 2.0]`, `[AdGuard 1.0]`, etc
  - Hint comments: `!+ NOT_OPTIMIZED`, etc
  - Metadata comments: `! Title: ...`, `! Homepage: ...`, etc
  - Regular comments: `! Comment`, `! Comment for the rule`, etc
- Cosmetic rules have the following types:
  - Element hiding rules: `##.ad`, `example.com##.ad`, etc
  - CSS injection rules: `#$#body { background: red; }`, etc
  - Scriptlet injection rules: `#%#//scriptlet(...)`, `##+js(...)`, etc
  - HTML filtering rules: `$$script[tag-content="..."]`
    `##^script:has-text(...)`, etc

#### Keep comments with the rule

When ordering rules, we should consider the following things:

- some rules have its own comments

  - comments should be kept with the rule (when moving a rule, its comments
    should be moved together with it)

    ```
    ! Comment
    rule
    ```

    or issue comments:

    ```
    ! https://github.com/yourname/yourrepo/issues/1
    rule
    ```

    or multiple comments:

    ```
    ! Comment for the rule + a link to issue
    ! https://github.com/yourname/yourrepo/issues/1
    rule
    ```

    but not:

    ```
    ! Random comment

    rule
    ```

    (because there is an empty / separator line between the comment and the
    rule)

Ordering rules in a block based on the following aspects:

- Rule category
  - Rule type within the same category
- Exceptionality (normal rules first, exception rules later)
- Specificity (general rules first, specific rules later)
  - _General rules_ are rules which have no domain specified, and applied to
    all domains
    - For example: `##.ad`, `/ads.js`, etc.
  - _Specific rules_ are rules which have domain specified, and applied to
    specific domains only
    - For example: `example.com##.ad`, `/ads.js$domain=example.com`, etc.

### Config file

We should allow users to specify the following things in the config file:
- Ordering aspects (see above)
- Separator line pattern (eg use `! ---` as the separator line, etc)

TODO

### API

- CLI tool
- Programmatic API

TODO