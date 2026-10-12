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
jyro man string/overview
```

## Topic resolution

The topic is compared with the slugs of the manual's pages, subsections, and sections. A slug is the title in lowercase with spaces replaced by hyphens, for example `types-and-values` and `string-concatenation`. The first of these rules that matches decides the result:

1. A page whose slug is the topic.
2. A subsection whose slug is the topic.
3. A section whose slug is the topic.
4. A single page whose slug starts with the topic, or contains it after a hyphen.
5. A single subsection whose slug matches the same way.

When rule 1, 4, or 5 matches several entries, the command lists them and exits with 1. When nothing matches, it prints a message and exits with 1.

## Paths

Several pages share a title: most sections and subsections have an Overview, and there is more than one page named Libraries, Diagnostics, and Resource Limits. A topic that contains `/` is a path, which names such a page by the slugs of the section and subsection that hold it.

```bash
jyro man string/overview
jyro man standard-library/string/overview
jyro man /docs/standard-library/string/overview
```

A path matches the page whose section, subsection, and page slugs end with the slugs of the path, so the leading slugs may be left out while the rest still names one page. A leading `/docs/`, the form of the manual's addresses on the website, is ignored. When no page matches, a path matches a subsection the same way, and then a section. A path that matches several pages lists them and exits with 1.

When a topic names more than one page, the list shows each by the shortest path that names it alone, and that path is the one the command accepts.

```bash
jyro man overview
```

```text
Ambiguous topic 'overview'. Did you mean:

  language/overview           Language
  standard-library/overview   Standard Library
  string/overview             Standard Library > String
  ...
```

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

When standard output is redirected, the page is written as text with a footer naming its section, the pages beside it, and the `jyro man` command for each, with a path where the title alone names more than one page. A subsection or section topic writes its index.
