# man

Shows this manual in the terminal.

```bash
jyro man [<topic> ...]
```

Without a topic, the command shows the table of contents. With a topic, it shows the matching page, subsection, or section. A topic of several words is joined with hyphens, and the lookup ignores case.

```bash
jyro man
jyro man foreach
jyro man "types and values"
jyro man standard-library
```

## Topic resolution

The topic is compared with the slugs of the manual's pages, subsections, and sections. A slug is the title in lowercase with spaces replaced by hyphens, for example `types-and-values` and `string-concatenation`. The first of these rules that matches decides the result:

1. A page whose slug is the topic.
2. A subsection whose slug is the topic.
3. A section whose slug is the topic.
4. A single page whose slug starts with the topic, or contains it after a hyphen.
5. A single subsection whose slug matches the same way.

When rule 4 or 5 matches several entries, the command lists them and exits with 1. When nothing matches, it prints a message and exits with 1.

## Interactive display

When standard output is a terminal, the manual opens in a browser inside the terminal.

| Key | Action |
| --- | --- |
| Up, Down, Page Up, Page Down, Home, End | Scroll |
| Tab, Shift+Tab | Move between links |
| Enter | Follow the selected link |
| Backspace, Esc | Go back |
| `q` | Quit |

The status bar shows the page title and the scroll position. The text reflows when the window is resized. A page is followed by its breadcrumb and the pages of its subsection.

## Redirected output

When standard output is redirected, the page is written as text with a footer naming its section, the pages beside it, and the `jyro man` command for each. A subsection or section topic writes its index.
