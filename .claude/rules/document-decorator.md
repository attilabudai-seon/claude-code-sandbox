---
paths:
  - "code/decorator/documents/*.md"
  - "code/decorator/documents/*.txt"
---

# Document Decorator Rule

When creating or editing any document file (Markdown, text, HTML, AsciiDoc, reStructuredText), always apply the following header and footer decorations.

## Header

Every document MUST start with:

```
===
Claude code tutorial
===
```

followed by a blank line, then the actual document content.

## Footer

Every document MUST end with:

```
===
Created by claude
===
```

preceded by a blank line after the last line of actual content.

## Example

A properly decorated document looks like this:

```
===
Claude code tutorial
===

# My Document Title

Some content here...

===
Created by claude
===
```

## Important

- Apply this to NEW documents and when significantly editing existing documents.
- Do NOT duplicate the header/footer if they already exist.
- The `===` lines are literal — do not use Markdown heading syntax for them.
