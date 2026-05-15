---
name: gitbook-widgets
description: Reference card for GitBook UI components used in this developer portal. Use when writing or editing docs pages and you need the correct syntax for hints, embeds, tabs, figures, collapsible sections, cards, or YAML front matter. Trigger on "hint block", "callout", "embed video", "tabs", "figure", "details", "collapsible", "front matter", "gitbook syntax", "how to add image".
tools: Read
---

# GitBook Widgets — Supervisely Developer Portal

Quick reference for every GitBook-flavored element found in this repo. Copy-paste ready.

---

## YAML Front Matter

Every page should start with a `description` front matter block. GitBook renders it as a subtitle under the page title.

```markdown
---
description: One-sentence description shown as page subtitle on developer.supervisely.com
---

# Page Title
```

---

## Hints (callout boxes)

Three styles used in this repo. Always close with `{% endhint %}`.

```markdown
{% hint style="info" %}
Informational note — tips, links to related resources, "learn more" pointers.
{% endhint %}

{% hint style="warning" %}
Caution — beta features, breaking changes, things that can go wrong.
{% endhint %}

{% hint style="success" %}
Positive outcome — confirmation that something works, highlight of a key benefit.
{% endhint %}
```

> `style="danger"` exists in GitBook but is **not used** in this repo. Use `warning` instead.

---

## Embedded media (`{% embed %}`)

Use for YouTube videos and other embeddable URLs. GitBook renders an inline player.

```markdown
{% embed url="https://youtu.be/VIDEO_ID" %}
{% endembed %}
```

For `.mp4` files hosted on GitHub releases:

```markdown
{% embed url="https://user-images.githubusercontent.com/ORG/REPO/releases/download/vX.Y.Z/demo.mp4" %}
{% endembed %}
```

---

## Tabs

Group related code samples or screenshots by variant.

```markdown
{% tabs %}
{% tab title="First Option" %}
Content for first tab — text, code blocks, images, anything.
{% endtab %}

{% tab title="Second Option" %}
Content for second tab.
{% endtab %}
{% endtabs %}
```

---

## Figures (images)

**External image (GitHub releases / user-images — preferred):**

```markdown
<figure><img src="https://user-images.githubusercontent.com/USER_ID/ASSET_ID.png" alt="Alt text"><figcaption><p>Caption text</p></figcaption></figure>
```

**Local asset (`.gitbook/assets/`):**

```markdown
<figure><img src="../../../.gitbook/assets/filename.png" alt="Alt text"><figcaption><p>Caption text</p></figcaption></figure>
```

Adjust the number of `../` to match the page's depth relative to repo root:
- `getting-started/*.md` → `../../.gitbook/assets/`
- `getting-started/python-sdk-tutorials/videos/*.md` → `../../../.gitbook/assets/` (would be going up 3 levels but .gitbook is at root — count slashes to root)
- `app-development/widgets/input/*.md` → `../../../.gitbook/assets/`

**Without caption** (omit `<p>` content but keep the tag):

```markdown
<figure><img src="URL" alt=""><figcaption></figcaption></figure>
```

**Sized image** (`data-size` controls display size):

```markdown
<figure><img src="URL" alt="" data-size="original"><figcaption></figcaption></figure>
```

---

## Collapsible sections (`<details>`)

Use for long code examples or supplementary content that shouldn't break the reading flow.

```markdown
<details>

<summary>Summary line shown when collapsed</summary>

Full content shown when expanded. Can include code blocks, text, images.

```python
# example code
```

</details>
```

---

## Highlighted / colored text (`<mark>`)

Used in card grids for widget descriptions. Not for regular prose.

```markdown
<mark style="color:green;">text</mark>
<mark style="color:purple;">text</mark>
```

---

## Card grid (`<table data-view="cards">`)

Used in widget index pages to display a visual card grid. Each row = one card.

```markdown
<table data-card-size="large" data-view="cards">
<thead><tr><th></th><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead>
<tbody>
<tr>
  <td><strong>Widget Name</strong></td>
  <td><img src="../../../.gitbook/assets/widget-name.png" alt=""></td>
  <td><mark style="color:purple;">Short description of what the widget does</mark></td>
  <td><a href="widget-name.md">widget-name.md</a></td>
</tr>
</tbody>
</table>
```

---

## Cross-page links

Use relative `.md` paths — GitBook rewrites to clean URLs at build time.

```markdown
[Link text](../other-section/page.md)
[Link to anchor](./same-dir-page.md#section-heading)
```

---

## Rules

- **New page** → must be added to `SUMMARY.md` at the correct indentation depth, or GitBook won't show it in the sidebar.
- **Renamed/moved page** → add a redirect in `.gitbook.yaml` under `redirects:` from old path to new `.md` path.
- **Images** → prefer external hosting (GitHub releases). Only use `.gitbook/assets/` for assets that don't belong to a GitHub release.
- **No `style="danger"` hints** — not in use in this repo.
