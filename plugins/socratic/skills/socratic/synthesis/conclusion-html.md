# Conclusion HTML Synthesis

FEYNMAN writes `conclusion.html`.

## Teaching Contract

Write as a technical blog post for a smart reader who knows the field but has not seen this design. The reader should be able to follow the article without opening the source code, paper, or previous session artifacts.

Requirements:

- Every causal step is explicit: "because", "which means", "so that".
- Every new term is glossed in the same sentence or dropped.
- Every paragraph includes a concrete example or source reference.
- Preserve the causal spine: situation -> pressure -> mechanism question -> answer. Do not force fixed SCQA section headings.
- Carry verification markers: `^v`, `^p`, `^u`, `^a`.
- Keep failure-list items visible; do not smooth over them.

## HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Feynman Synthesis — {topic}</title>
  <style>
    /* readable body font, monospace for code, color-coded verification markers */
  </style>
</head>
<body>
  <header>
    <h1>Feynman Synthesis — {topic}</h1>
    <p class="meta">Session: {session-path or cold draft directory} · Generated: {date} · Source: {warm|cold}</p>
    <!-- Cold path only: visible warning that this draft has no prior grill/evidence base. -->
  </header>

  <article>
    <!-- Blog-style teaching document. Use headings that fit the topic. -->
  </article>

  <section id="failure-list">
    <h2>What could not be explained simply</h2>
    <!-- F1/F2/F3 items with user resolution notes, or a note that none remain. -->
  </section>

  <section id="evidence">
    <h2>Evidence</h2>
    <!-- Citations to evidence files; code blocks followed by plain-language interpretation. -->
  </section>

  <footer>
    <p>Verification: {count ^v} verified · {count ^p} user-asserted · {count ^u} unverified · {count ^a} assumption</p>
  </footer>
</body>
</html>
```

Verification markers render as superscripts:

| Marker | Color |
|---|---|
| `^v` | green |
| `^p` | blue |
| `^u` | orange |
| `^a` | red |

After writing, print a 3-line summary: file path, verification score (`^v` / all claims), and open failure-list count.
