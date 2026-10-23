# Comments

TugQL has two forms, and each has its own comment syntax. The file name decides
which form a query body is: `<id>.query.tugql` is the text form and
`<id>.query.tugql.yaml` is the structured YAML form.

## Text form

`--` and `//` both start an end-of-line comment. The comment runs to the end of
the line.

```sql
SELECT id, name   -- SQL-style comment
FROM customers    // C-style comment, familiar from most programming languages
WHERE active = true
```

- A marker inside a string literal or a quoted identifier is not a comment
  (`'a -- b'`, `'http://example.com'`, `"odd // name"`).
- `#` is **not** a comment in text form.

## YAML form

`#` is the comment marker, as in any YAML document.

```yaml
from: customers   # a YAML comment
where: active = true
```

## Conversion between the forms

- Text to YAML: both `--` and `//` comments become `#` comments.
- YAML to text: `#` comments are written as `--` comments.

The `//` marker is therefore lost on a text to YAML to text round trip: a
`//` comment comes back as `--`. The comment text is preserved; only the
marker changes. This loss is documented and accepted.

Comments are never dropped silently, in either direction. A comment with no
clean anchor becomes the leading comment of the nearest following element. A
comment that trails the document becomes a document-end comment. If neither
placement works, a positioned diagnostic is raised instead of losing the
comment.

The formatter keeps the marker the author wrote (`--` stays `--`, `//` stays
`//`) when it reformats a text-form query.
