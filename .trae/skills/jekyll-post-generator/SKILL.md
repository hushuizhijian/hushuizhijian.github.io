---
name: "jekyll-post-generator"
description: "Generates Jekyll blog posts conforming to the Hux Blog template format. Invoke when user wants to create a new blog post, generate Markdown files for Jekyll, or batch-produce standardized posts."
---

# Jekyll Post Generator

This Skill generates Jekyll blog post Markdown files that strictly conform to the Hux Blog template format (Jekyll + kramdown + GFM). It ensures YAML frontmatter compliance, Markdown body formatting consistency, and static site rendering compatibility.

## 1. Trigger Rules

Invoke this Skill when:
- User asks to create a new blog post / Markdown file for the Jekyll site
- User asks to generate, write, or batch-produce posts conforming to the existing template
- User provides raw content and wants it formatted into a standard Jekyll post
- User mentions "new post", "write article", "generate post", "batch generate posts"

## 2. Input Parameters

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| `title` | YES | Post title (Chinese or English) | `"如何通俗地解释停机问题？"` |
| `content` | YES | Raw text / content to be formatted into the post body | Any text |
| `date` | YES | Publication date, format `YYYY-MM-DD` | `"2026-04-30"` |
| `subtitle` | NO | Subtitle (English translation or supplement) | `"How to explain the Halting Problem?"` |
| `tags` | YES | Tag list (YAML array) | `[Web, 知乎]` |
| `author` | NO | Author name, default `"Hux"` | `"Hux"` |
| `header-img` | NO | Header image path, quoted string | `"img/post-bg-web.jpg"` |
| `header-mask` | NO | Header overlay opacity (0.0-1.0), required if `header-img` is set | `0.4` |
| `header-style` | NO | Header style, value `text` when no image header | `text` |
| `catalog` | NO | Whether to show sidebar catalog, boolean | `true` |
| `published` | NO | Whether the post is published, boolean | `true` |
| `mathjax` | NO | Whether to enable MathJax rendering, boolean | `true` |
| `slug` | NO | URL slug (auto-derived from title if omitted) | `"halting-problem"` |
| `source-url` | NO | Original source URL (for reposted content) | `"https://www.zhihu.com/..."` |
| `source-desc` | NO | Source description text | `"我在知乎上的回答"` |

## 3. Output Logic

### 3.1 File Naming

File MUST be named: `YYYY-MM-DD-slug.md` (or `.markdown`)

Rules:
- Date prefix matches the `date` parameter
- Slug is auto-derived from title: lowercase, spaces replaced by hyphens, remove special characters except hyphens
- Example: title `"如何通俗地解释停机问题？"` → slug `halting-problem` → filename `2017-12-12-halting-problem.md`

### 3.2 YAML Frontmatter Generation

The YAML frontmatter MUST be generated according to these rules:

```
---
layout: post
title: "<title_value>"
subtitle: "<subtitle_value>"
author: "<author_value>"
header-img: "<header_img_value>"
header-mask: <header_mask_value>
tags:
  - <tag1>
  - <tag2>
---
```

**Field generation logic:**

1. `layout` — ALWAYS `post`, unquoted
2. `title` — ALWAYS double-quoted, even if no special characters
3. `subtitle` — double-quoted if present; omit field entirely if not provided
4. `author` — ALWAYS double-quoted, default `"Hux"`
5. `header-img` — double-quoted string; if no image, use `header-style: text` instead
6. `header-mask` — decimal number (unquoted), only present when `header-img` is set
7. `header-style` — unquoted string `text`, only present when NO `header-img`
8. `catalog` — boolean (unquoted `true`/`false`), omit if not specified
9. `published` — boolean (unquoted `true`/`false`), omit if not specified (default: published)
10. `mathjax` — boolean (unquoted `true`), only present when post contains LaTeX math
11. `tags` — YAML block sequence, one `- ` per line, 2-space indent under `tags:`
12. `date` — format `YYYY-MM-DD HH:MM:SS`, only include if explicitly provided

**Fields that are mutually exclusive:**
- `header-img` + `header-mask` vs. `header-style: text` (use one or the other, never both)

### 3.3 Markdown Body Generation

#### 3.3.1 Opening Blockquote

If the post is reposted from another source (`source-url` provided), start with:

```
> 这篇文章转载自[<source-desc>](<source-url>)
```

If the post is original, start with:

```
> <a brief opening quote or statement>
```

Leave a blank line after the blockquote.

#### 3.3.2 Heading Hierarchy

- `##` (ATX H2) for major sections: `## 背景知识`, `## 正文`, `## 后记`, `## 意义`, `## 漫谈`
- `###` (ATX H3) for subsections
- Setext-style H2 (`text` + `--` on next line) is acceptable for informal section breaks like `引言`, `补充`, `参考`
- Setext-style H1 (`text` + `==` on next line) is acceptable for prominent section titles
- NEVER use `#` (H1) in ATX style — the post title is already H1
- Maximum heading depth: H4 (`####`)

#### 3.3.3 Paragraphs and Line Breaks

- Blank line between paragraphs
- Two trailing spaces at end of line for hard line breaks within a paragraph (visible line break without new paragraph)
- Do NOT use `<br>` tags unless absolutely necessary (e.g., to create spacing around math blocks)

#### 3.3.4 Emphasis

- Bold: `**text**` (double asterisks, NOT `__text__`)
- Italic: `_text_` (single underscores for emphasis, NOT `*text*`)
- Bold + Italic: `**_text_**`

#### 3.3.5 Code

- Inline code: `` `code` `` (single backticks)
- Fenced code blocks: triple backticks with language identifier

```
```python
def hello():
    print("world")
```
```

Supported language identifiers: `python`/`py`, `javascript`/`js`, `typescript`/`ts`, `css`, `html`, `bash`/`sh`, `json`, `yaml`, `ruby`, `go`, `rust`, `java`, `c`, `cpp`

#### 3.3.6 Links

- Inline link: `[link text](URL)`
- Reference-style link is acceptable for repeated URLs
- Auto-link for bare URLs: `<https://example.com>`

#### 3.3.7 Images

- Standard: `![](image-url)` — alt text may be omitted
- With alt text: `![alt text](image-url)`
- Local images: `![](/img/in-post/<folder>/<filename>)`
- External images: `![](https://...)`
- Images MUST be on their own line, NOT inline with text

#### 3.3.8 Lists

- Unordered: use `* ` (asterisk + space) as primary; `- ` (hyphen + space) is also acceptable
- Ordered: use `1. ` (number + period + space); increment numbers logically
- Nested: 4-space indent per level (or 2-space if following existing convention in the file)
- List item continuation: indent paragraph text to align with list item text

#### 3.3.9 Blockquotes

- Single line: `> text`
- Multi-line: `> ` prefix on each line
- Nested: `>> ` for second level
- Leave blank line before and after blockquote

#### 3.3.10 Tables (GFM Extension)

Standard GFM pipe table syntax:

```
| Header 1 | Header 2 | Header 3 |
| -------- | -------- | -------- |
| Cell 1   | Cell 2   | Cell 3   |
```

- Alignment row uses hyphens, at least 3 per cell
- Alignment: `:---` left, `:---:` center, `---:` right

#### 3.3.11 Horizontal Rules

- Three hyphens `---` on their own line
- Use to separate major content sections

#### 3.3.12 Math (Extension — requires `mathjax: true`)

- Inline math: `$expression$`
- Display math: `$$expression$$` on its own line
- Only use when `mathjax: true` is set in frontmatter

#### 3.3.13 Footnotes / References

Use the pattern `[\[N\]](#ref_N)` for reference markers in text, and a numbered list at the end:

```
参考
--

1.  Description URL
2.  Description URL
```

#### 3.3.14 Closing

End the post body naturally. Common patterns:
- `以上。` (for Q&A style posts)
- A brief closing statement or date signature

## 4. Format Validation Rules

After generating a Markdown file, perform the following validation checks. If any check fails, provide a specific correction message and fix the issue.

### 4.1 YAML Frontmatter Validation

| # | Check | Pass Criteria | Fix Message |
|---|-------|---------------|-------------|
| Y1 | Frontmatter delimiters | Starts with `---` on line 1, ends with `---` on its own line | "YAML frontmatter must be delimited by `---` on separate lines" |
| Y2 | `layout` field present | `layout: post` exists | "Missing required field `layout: post`" |
| Y3 | `title` field present | `title:` exists and value is double-quoted | "Missing required field `title`, or value not double-quoted" |
| Y4 | `author` field present | `author:` exists and value is double-quoted | "Missing required field `author`, or value not double-quoted" |
| Y5 | `tags` field present | `tags:` exists as YAML block sequence | "Missing required field `tags`" |
| Y6 | Tags format | Each tag on its own line with `- ` prefix, 2-space indent | "Tags must use YAML block sequence format with 2-space indent" |
| Y7 | String quoting | All string values containing `:`, `#`, `{`, `}`, `[`, `]`, `,`, `&`, `*`, `?`, `|`, `-`, `<`, `>`, `=`, `!`, `%`, `@`, `` ` `` are double-quoted | "String value contains special characters and must be double-quoted" |
| Y8 | Boolean format | `true`/`false` unquoted, NOT `"true"`/`"false"` | "Boolean values must be unquoted lowercase" |
| Y9 | Numeric format | Numbers unquoted | "Numeric values must be unquoted" |
| Y10 | Colon spacing | Every `key: value` pair has a space after the colon | "YAML key-value pair must have a space after colon" |
| Y11 | Consistent indent | No mixed tabs/spaces; consistent 2-space indentation | "YAML indentation must use consistent 2-space indentation, no tabs" |
| Y12 | header-img/header-style exclusivity | Not both `header-img` and `header-style: text` present simultaneously | "Cannot have both `header-img` and `header-style: text`; choose one" |
| Y13 | header-mask with header-img | If `header-img` is set, `header-mask` should also be set | "When `header-img` is set, `header-mask` should also be specified" |

### 4.2 Markdown Body Validation

| # | Check | Pass Criteria | Fix Message |
|---|-------|---------------|-------------|
| M1 | No H1 ATX heading | No `# ` heading in body (title is already H1) | "Do not use `#` (H1) ATX heading in post body; start from `##`" |
| M2 | Code block fencing | All fenced code blocks have matching opening and closing triple backticks | "Unmatched fenced code block — ensure opening and closing ``` match" |
| M3 | Code block language | Fenced code blocks specify a language identifier | "Fenced code block should specify a language identifier after ```" |
| M4 | Image syntax | All images use `![...](...)` or `![](...)` format | "Image syntax error — use `![alt](url)` or `![](url)`" |
| M5 | Link syntax | All links use `[text](url)` format, no bare URLs in text | "Link syntax error — use `[text](url)` format" |
| M6 | Blank line before headings | At least one blank line before any heading | "Insert blank line before heading" |
| M7 | Blank line after headings | At least one blank line after any heading | "Insert blank line after heading" |
| M8 | No HTML in Markdown | No raw HTML tags except `<br>`, `<p id="...">`, and inline elements | "Avoid raw HTML in Markdown; use Markdown syntax instead" |
| M9 | MathJax consistency | If `$...$` or `$$...$$` appears, `mathjax: true` must be in frontmatter | "Post contains LaTeX math but `mathjax: true` is not set in frontmatter" |
| M10 | Table formatting | Pipe tables have header row, separator row, and body rows | "Table formatting error — ensure header, separator, and body rows exist" |
| M11 | Trailing whitespace for line breaks | Hard line breaks use two trailing spaces, NOT `\` | "Use two trailing spaces for hard line breaks within paragraphs" |

### 4.3 File-Level Validation

| # | Check | Pass Criteria | Fix Message |
|---|-------|---------------|-------------|
| F1 | Filename format | Matches `YYYY-MM-DD-slug.md` or `.markdown` | "Filename must follow `YYYY-MM-DD-slug.md` pattern" |
| F2 | Date consistency | Date in filename matches `date` field in frontmatter (if present) | "Date in filename must match `date` field in frontmatter" |
| F3 | File extension | `.md` or `.markdown` | "File extension must be `.md` or `.markdown`" |
| F4 | UTF-8 encoding | File is valid UTF-8 | "File must be UTF-8 encoded" |
| F5 | Trailing newline | File ends with a single newline character | "File must end with a trailing newline" |

## 5. YAML Format Checklist

When generating or reviewing the YAML frontmatter, verify each item:

- [ ] **YAML 1.2 compliant**: All syntax follows YAML 1.2 specification
- [ ] **Delimiter**: Frontmatter starts and ends with `---` on its own line
- [ ] **Key naming**: All keys are lowercase with hyphens (e.g., `header-img`, `header-style`)
- [ ] **Colon + space**: Every `key: value` pair has exactly one space after the colon
- [ ] **String quoting**: Strings containing special characters (`: # { } [ ] , & * ? | - < > = ! % @`) are double-quoted
- [ ] **Title quoting**: `title` value is ALWAYS double-quoted regardless of content
- [ ] **Author quoting**: `author` value is ALWAYS double-quoted
- [ ] **Boolean values**: `true`/`false` are unquoted, lowercase
- [ ] **Numeric values**: Numbers (including `header-mask` decimals) are unquoted
- [ ] **Tags format**: Uses YAML block sequence (`- item`) with 2-space indentation
- [ ] **No tabs**: Indentation uses spaces only, never tabs
- [ ] **Consistent indentation**: All nested content uses consistent 2-space indentation
- [ ] **No trailing whitespace** on YAML lines (except within quoted strings)
- [ ] **Date format**: If `date` field is present, format is `YYYY-MM-DD HH:MM:SS`
- [ ] **Required fields complete**: `layout`, `title`, `author`, `tags` are all present
- [ ] **Mutual exclusivity**: `header-img` and `header-style: text` are not both present
- [ ] **Conditional fields**: `header-mask` is present when `header-img` is set; `mathjax` is present when math is used

## 6. Usage Examples

### Example 1: Simple Text Post (No Image Header)

**Input:**
```
title: "为什么 CSS 这么难学？"
subtitle: "Why I dislike CSS as a programming language"
date: "2017-10-06"
tags: [Web, CSS, 知乎]
header-style: text
source-url: "https://www.zhihu.com/question/66167982/answer/240434582"
source-desc: "我在知乎上的回答"
content: |
  CSS 难学是因为它「出乎我意料之外的复杂」且让我觉得「定位矛盾」。
  CSS 的属性互不正交，大量的依赖与耦合难以记忆。
  ...
```

**Output file: `2017-10-06-css-complaints.md`**

```markdown
---
layout: post
title: "为什么 CSS 这么难学？"
subtitle: "Why I dislike CSS as a programming language"
author: "Hux"
header-style: text
tags:
  - Web
  - CSS
  - 知乎
---

> 这篇文章转载自[我在知乎上的回答](https://www.zhihu.com/question/66167982/answer/240434582)

CSS 难学是因为它**「出乎我意料之外的复杂」**且让我觉得**「定位矛盾」**。

CSS 的属性互不正交，大量的依赖与耦合难以记忆。

...
```

### Example 2: Post with Image Header and Code Blocks

**Input:**
```
title: "如何通俗地解释停机问题？"
subtitle: "How to explain the Halting Problem?"
date: "2017-12-12"
tags: [知乎, 计算理论]
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
content: |
  停机问题研究的是：是否存在一个程序，能够判断另外一个程序...
  (contains Python code examples)
```

**Output file: `2017-12-12-halting-problem.md`**

```markdown
---
layout: post
title: "如何通俗地解释停机问题？"
subtitle: "How to explain the Halting Problem?"
author: "Hux"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
tags:
  - 知乎
  - 计算理论
---

停机问题研究的是：是否存在一个「程序」，能够判断另外一个「程序」在特定的「输入」下...

```python
def is_halt(program, input) -> bool:
    # 返回 True  如果 program(input) 会返回
    # 返回 False 如果 program(input) 不返回
```

...
```

### Example 3: Post with MathJax

**Input:**
```
title: "如何证明不可计算的函数比可计算的函数多？"
subtitle: "Why is there more uncomputable functions?"
date: "2017-12-12"
tags: [知乎, 计算理论]
header-img: "img/post-bg-infinity.jpg"
header-mask: 0.3
mathjax: true
content: |
  使用形式语言来证明...
  A = { <G> | G is a connected undirected graph }
```

**Output file: `2017-12-12-uncomputable-funcs.md`**

```markdown
---
layout: post
title: "如何证明不可计算的函数比可计算的函数多？"
subtitle: "Why is there more uncomputable functions?"
author: "Hux"
header-img: "img/post-bg-infinity.jpg"
header-mask: 0.3
mathjax: true
tags:
  - 知乎
  - 计算理论
---

使用形式语言来证明：

$$
A = \{ \langle G \rangle \vert G \text{ is a connected undirected graph}\}
$$

...
```

### Example 4: Batch Generation

When user provides multiple content items, generate each as a separate file:

**Input:**
```
Generate 3 posts:
1. title: "Web 性能优化指南", date: "2026-05-01", tags: [Web, 性能]
2. title: "React Server Components 详解", date: "2026-05-02", tags: [React, Web]
3. title: "TypeScript 5.0 新特性", date: "2026-05-03", tags: [TypeScript, Web]
```

**Output:** Three files generated:
- `2026-05-01-web-performance-guide.md`
- `2026-05-02-react-server-components.md`
- `2026-05-03-typescript-5-features.md`

Each file undergoes full format validation before output.

## 7. Batch Generation Protocol

When generating multiple posts:

1. Process each post independently through the full generation + validation pipeline
2. Collect all validation results
3. If any post fails validation, report ALL failures together (do not stop at first failure)
4. Fix all validation issues and re-validate
5. Output all files only after ALL posts pass validation
6. Provide a summary: `Generated N/N posts successfully. Files: [list]`

## 8. Extension Syntax Restrictions

The following extended Markdown syntax is permitted ONLY in the specified contexts:

| Syntax | Context | Requirement |
|--------|---------|-------------|
| GFM Tables | Data comparison, feature matrices | Standard pipe table syntax only |
| LaTeX Math (`$...$`, `$$...$$`) | Mathematical content | Must set `mathjax: true` in frontmatter |
| Footnotes (`[^1]`) | Academic or reference-heavy posts | Use `[\[N\]](#ref_N)` pattern instead for this template |
| Task lists (`- [ ]`) | Not used in this template | Avoid |
| Strikethrough (`~~text~~`) | Casual content only | Avoid in formal posts |
| Auto-links (`<URL>`) | Bare URLs that need to be clickable | Prefer `[text](URL)` format |

All other non-standard Markdown extensions are PROHIBITED.
