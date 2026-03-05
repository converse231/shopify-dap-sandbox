# DAP Sandbox — Development Context

This is a sandbox environment built on the **Horizon** Shopify theme (v3.4.0) for developing custom sections for client stores. The theme uses Shopify Online Store 2.0 with JSON templates, a block-based architecture, and CSS custom properties for design tokens.

---

## Theme Architecture

### Directory Structure

```
dap-sandbox/
├── assets/          # CSS, JS modules, SVG icons
├── blocks/          # Block definitions (reusable components inside sections)
├── config/          # Theme settings (settings_schema.json, settings_data.json)
├── layout/          # Layout templates (theme.liquid, password.liquid)
├── locales/         # Translation files (en.default.json, *.schema.json)
├── sections/        # Section templates (page building blocks)
├── snippets/        # Reusable Liquid partials (shared UI and logic)
└── templates/       # Page templates (index.json, product.json, etc.)
```

### Layout Flow

```
theme.liquid
├── <head>
│   ├── snippets/meta-tags
│   ├── snippets/stylesheets      → preloads base.css
│   ├── snippets/fonts             → preloads woff2 font files
│   ├── snippets/scripts           → import map + JS modules
│   ├── snippets/theme-styles-variables → CSS custom properties
│   ├── snippets/color-schemes     → color scheme CSS variables
│   └── content_for_header         → Shopify-injected scripts
├── <body>
│   ├── #header-group              → sections 'header-group'
│   ├── <main> content_for_layout  → page-specific sections
│   ├── <footer> sections 'footer-group'
│   ├── snippets/search-modal
│   └── snippets/quick-add-modal   (if enabled)
```

### Template → Section → Block Flow

Templates (JSON) define which sections appear on a page and in what order. Each section renders its own blocks via `{% content_for 'blocks' %}`. Blocks are the atomic units of content.

```
templates/index.json
  → sections/hero.liquid
      → blocks/text.liquid (heading)
      → blocks/text.liquid (body)
      → blocks/button.liquid
  → sections/section.liquid
      → blocks/group.liquid
          → blocks/icon.liquid
          → blocks/text.liquid
```

---

## Creating a New Section

### Step 1: Create the Section File

Create `sections/my-section.liquid`. Use the standard capture-and-render pattern:

```liquid
{% capture children %}
  {% content_for 'blocks' %}
{% endcapture %}

{% render 'section', section: section, children: children %}

{% schema %}
{
  "name": "My Section",
  "class": "section-wrapper",
  "blocks": [
    { "type": "@theme" },
    { "type": "@app" }
  ],
  "settings": [ ... ],
  "presets": [ ... ]
}
{% endschema %}
```

The `snippets/section.liquid` wrapper provides:
- `.section-background` with color scheme
- `.section` container with width/height
- Background media (image/video) via `snippets/background-media`
- Border styling via `snippets/border-override`
- Optional overlay via `snippets/overlay`
- Flex layout container with `layout-panel-flex` class
- Responsive padding via `snippets/spacing-style`

### Step 2: Define the Schema

Use the standard settings order. Here is a minimal but complete template:

```json
{
  "name": "My Section",
  "class": "section-wrapper",
  "blocks": [
    { "type": "@theme" },
    { "type": "@app" }
  ],
  "disabled_on": {
    "groups": ["header"]
  },
  "settings": [
    {
      "type": "header",
      "content": "t:content.layout"
    },
    {
      "type": "select",
      "id": "content_direction",
      "label": "t:settings.direction",
      "options": [
        { "value": "column", "label": "t:options.vertical" },
        { "value": "row", "label": "t:options.horizontal" }
      ],
      "default": "column"
    },
    {
      "type": "checkbox",
      "id": "vertical_on_mobile",
      "label": "t:settings.vertical_on_mobile",
      "default": true,
      "visible_if": "{{ section.settings.content_direction == 'row' }}"
    },
    {
      "type": "range",
      "id": "gap",
      "label": "t:settings.gap",
      "min": 0,
      "max": 100,
      "step": 1,
      "unit": "px",
      "default": 12
    },
    {
      "type": "header",
      "content": "t:content.size"
    },
    {
      "type": "select",
      "id": "section_width",
      "label": "t:settings.width",
      "options": [
        { "value": "page-width", "label": "t:options.page" },
        { "value": "full-width", "label": "t:options.full" }
      ],
      "default": "page-width"
    },
    {
      "type": "select",
      "id": "section_height",
      "label": "t:settings.height",
      "options": [
        { "value": "", "label": "t:options.auto" },
        { "value": "small", "label": "t:options.small" },
        { "value": "medium", "label": "t:options.medium" },
        { "value": "large", "label": "t:options.large" },
        { "value": "full-screen", "label": "t:options.full_screen" },
        { "value": "custom", "label": "t:options.custom" }
      ],
      "default": ""
    },
    {
      "type": "range",
      "id": "section_height_custom",
      "label": "t:settings.custom_height",
      "min": 0,
      "max": 100,
      "step": 1,
      "default": 50,
      "unit": "%",
      "visible_if": "{{ section.settings.section_height == 'custom' }}"
    },
    {
      "type": "header",
      "content": "t:content.appearance"
    },
    {
      "type": "color_scheme",
      "id": "color_scheme",
      "label": "t:settings.color_scheme",
      "default": "scheme-1"
    },
    {
      "type": "select",
      "id": "background_media",
      "label": "t:settings.background_media",
      "options": [
        { "value": "none", "label": "t:options.none" },
        { "value": "image", "label": "t:options.image" },
        { "value": "video", "label": "t:options.video" }
      ],
      "default": "none"
    },
    {
      "type": "image_picker",
      "id": "background_image",
      "label": "t:settings.image",
      "visible_if": "{{ section.settings.background_media == 'image' }}"
    },
    {
      "type": "select",
      "id": "background_image_position",
      "label": "t:settings.image_position",
      "options": [
        { "value": "cover", "label": "t:options.cover" },
        { "value": "fit", "label": "t:options.fit" }
      ],
      "default": "cover",
      "visible_if": "{{ section.settings.background_media == 'image' }}"
    },
    {
      "type": "video",
      "id": "video",
      "label": "t:settings.video",
      "visible_if": "{{ section.settings.background_media == 'video' }}"
    },
    {
      "type": "select",
      "id": "video_position",
      "label": "t:settings.video_position",
      "options": [
        { "value": "cover", "label": "t:options.cover" },
        { "value": "contain", "label": "t:options.contain" }
      ],
      "default": "cover",
      "visible_if": "{{ section.settings.background_media == 'video' }}"
    },
    {
      "type": "select",
      "id": "border",
      "label": "t:settings.borders",
      "options": [
        { "value": "none", "label": "t:options.none" },
        { "value": "solid", "label": "t:options.solid" }
      ],
      "default": "none"
    },
    {
      "type": "range",
      "id": "border_width",
      "min": 0, "max": 10, "step": 0.5, "unit": "px",
      "label": "t:settings.border_width",
      "default": 1,
      "visible_if": "{{ section.settings.border != 'none' }}"
    },
    {
      "type": "range",
      "id": "border_opacity",
      "min": 0, "max": 100, "step": 1, "unit": "%",
      "label": "t:settings.border_opacity",
      "default": 100,
      "visible_if": "{{ section.settings.border != 'none' }}"
    },
    {
      "type": "range",
      "id": "border_radius",
      "label": "t:settings.border_radius",
      "min": 0, "max": 100, "step": 1, "unit": "px",
      "default": 0
    },
    {
      "type": "checkbox",
      "id": "toggle_overlay",
      "label": "t:settings.background_overlay"
    },
    {
      "type": "color",
      "id": "overlay_color",
      "label": "t:settings.overlay_color",
      "alpha": true,
      "default": "#00000026",
      "visible_if": "{{ section.settings.toggle_overlay }}"
    },
    {
      "type": "select",
      "id": "overlay_style",
      "label": "t:settings.overlay_style",
      "options": [
        { "value": "solid", "label": "t:options.solid" },
        { "value": "gradient", "label": "t:options.gradient" }
      ],
      "default": "solid",
      "visible_if": "{{ section.settings.toggle_overlay }}"
    },
    {
      "type": "select",
      "id": "gradient_direction",
      "label": "t:settings.gradient_direction",
      "options": [
        { "value": "to top", "label": "t:options.up" },
        { "value": "to bottom", "label": "t:options.down" }
      ],
      "default": "to top",
      "visible_if": "{{ section.settings.toggle_overlay and section.settings.overlay_style == 'gradient' }}"
    },
    {
      "type": "header",
      "content": "t:content.padding"
    },
    {
      "type": "range",
      "id": "padding-block-start",
      "label": "t:settings.top",
      "min": 0, "max": 100, "step": 1, "unit": "px",
      "default": 48
    },
    {
      "type": "range",
      "id": "padding-block-end",
      "label": "t:settings.bottom",
      "min": 0, "max": 100, "step": 1, "unit": "px",
      "default": 48
    }
  ],
  "presets": [
    {
      "name": "My Section",
      "category": "t:categories.text",
      "blocks": {
        "heading": {
          "type": "text",
          "settings": {
            "text": "<h2>Your heading</h2>",
            "type_preset": "h2"
          }
        },
        "body": {
          "type": "text",
          "settings": {
            "text": "<p>Your body text here.</p>"
          }
        },
        "button": {
          "type": "button",
          "settings": {
            "label": "Shop now",
            "link": "shopify://collections/all"
          }
        }
      },
      "block_order": ["heading", "body", "button"],
      "settings": {
        "content_direction": "column",
        "gap": 16,
        "section_width": "page-width",
        "padding-block-start": 48,
        "padding-block-end": 48
      }
    }
  ]
}
```

### Step 3: Add Custom Blocks (Optional)

If the section needs unique block types beyond the standard `@theme` blocks, create them in `blocks/`:

```liquid
{%- comment -%} blocks/my-custom-block.liquid {%- endcomment -%}

<div {{ block.shopify_attributes }}>
  {{ block.settings.text }}
</div>

{% schema %}
{
  "name": "My Custom Block",
  "settings": [
    {
      "type": "richtext",
      "id": "text",
      "label": "Text"
    }
  ]
}
{% endschema %}
```

Then reference it in the section schema:

```json
"blocks": [
  { "type": "@theme" },
  { "type": "@app" },
  { "type": "my-custom-block" }
]
```

### Step 4: Section-Specific CSS

Use `{% stylesheet %}` for scoped section CSS. Use theme CSS variables — never hardcode colors or fonts.

```liquid
{% stylesheet %}
  .my-section__title {
    font-family: var(--font-heading--family);
    color: var(--color-foreground);
    margin-block-end: var(--gap-sm);
  }

  @media screen and (max-width: 749px) {
    .my-section__title {
      font-size: var(--font-h3--size);
    }
  }
{% endstylesheet %}
```

### Step 5: Custom Section (Without Section Snippet)

For sections that need full control over their HTML structure (like the Hero), skip `snippets/section.liquid` and build the wrapper manually. Use the same utility snippets:

```liquid
<div
  class="my-section color-{{ section.settings.color_scheme }}"
  style="{% render 'spacing-style', settings: section.settings %}"
>
  <div
    class="
      my-section__content
      layout-panel-flex
      layout-panel-flex--{{ section.settings.content_direction }}
      section--{{ section.settings.section_width }}
      {% if section.settings.vertical_on_mobile %}mobile-column{% endif %}
    "
    style="{% render 'layout-panel-style', settings: section.settings %}"
  >
    {% content_for 'blocks' %}
  </div>
</div>
```

---

## Design Token Reference

All design tokens are defined as CSS custom properties in `snippets/theme-styles-variables.liquid` and `config/settings_schema.json`.

### Color Scheme Variables

Applied via the class `color-{{ scheme_id }}`. Each scheme provides:

| Variable | Purpose |
|----------|---------|
| `--color-background` | Section/block background |
| `--color-foreground` | Default text color |
| `--color-foreground-heading` | Heading text color |
| `--color-primary` | Links, accents |
| `--color-primary-hover` | Primary hover state |
| `--color-border` | Border color |
| `--color-border-rgb` | Border color as RGB (for opacity) |
| `--color-shadow` | Shadow color |

Available schemes: `scheme-1` through `scheme-6` (configured in `config/settings_data.json`).

### Typography Variables

| Variable | Purpose |
|----------|---------|
| `--font-body--family` | Body text font |
| `--font-heading--family` | Heading font |
| `--font-accent--family` | Accent/decorative font |
| `--font-subheading--family` | Subheading font |
| `--font-h1--size` ... `--font-h6--size` | Heading sizes (fluid with `clamp()`) |
| `--font-paragraph--size` | Body text size |

Type presets available for blocks: `h1`, `h2`, `h3`, `h4`, `h5`, `h6`, `paragraph`, `rte`, `custom`.

### Spacing Variables

| Variable Range | Purpose |
|----------------|---------|
| `--margin-3xs` ... `--margin-6xl` | Margin scale |
| `--padding-3xs` ... `--padding-6xl` | Padding scale |
| `--gap-3xs` ... `--gap-3xl` | Gap scale |
| `--spacing-scale` | Responsive multiplier (scales values > 20px) |
| `--gap-scale` | Responsive gap multiplier (scales values > 24px) |

### Section Height Variables

| Variable | Value |
|----------|-------|
| `--section-height-small` | Theme-configured small height |
| `--section-height-medium` | Theme-configured medium height |
| `--section-height-large` | Theme-configured large height |

### Z-Index Layers

| Variable | Value | Purpose |
|----------|-------|---------|
| `--layer-section-background` | -2 | Section backgrounds |
| `--layer-base` | 0 | Default layer |
| `--layer-flat` | 1 | Content above background |
| `--layer-raised` | 2 | Elevated elements |
| `--layer-sticky` | 5 | Sticky headers |
| `--layer-overlay` | 10 | Overlays, modals |
| `--layer-temporary` | 20 | Tooltips, popovers |

### Animation Variables

| Variable | Purpose |
|----------|---------|
| `--animation-speed-fast` | Fast transitions |
| `--animation-speed` | Default transition speed |
| `--animation-speed-slow` | Slow transitions |
| `--ease-out-cubic` | Cubic ease-out curve |
| `--ease-out-quad` | Quadratic ease-out curve |

### Page Width Variables

| Variable | Purpose |
|----------|---------|
| `--narrow-page-width` | 90rem |
| `--normal-page-width` | 120rem |
| `--wide-page-width` | 150rem |

---

## Snippet API Reference

### `section` — Section Wrapper

Renders the complete section wrapper with background, layout, overlay.

```liquid
{% render 'section', section: section, children: children %}
```

| Param | Type | Description |
|-------|------|-------------|
| `section` | section object | The current section |
| `children` | string | Captured block output |

Reads from `section.settings`: `section_width`, `section_height`, `section_height_custom`, `color_scheme`, `content_direction`, `vertical_on_mobile`, `background_media`, `background_image`, `background_image_position`, `video`, `video_position`, `border`, `border_width`, `border_opacity`, `border_radius`, `toggle_overlay`, `overlay_color`, `overlay_style`, `gradient_direction`, `padding-block-start`, `padding-block-end`.

### `group` — Block Container

Renders a flex container for nested blocks.

```liquid
{% render 'group', children: children, settings: block.settings, shopify_attributes: block.shopify_attributes %}
```

| Param | Type | Description |
|-------|------|-------------|
| `children` | string | Captured nested block output |
| `settings` | object | Block settings |
| `shopify_attributes` | string | Shopify editor attributes |

### `spacing-style` — Responsive Padding

Outputs CSS custom property declarations for padding. Values above 20px scale responsively.

```liquid
<div class="spacing-style" style="{% render 'spacing-style', settings: section.settings %}">
```

Reads: `padding-block-start`, `padding-block-end`, `padding-inline-start`, `padding-inline-end`.

### `layout-panel-style` — Flex Layout

Outputs CSS custom properties for flex direction, alignment, and gap.

```liquid
<div style="{% render 'layout-panel-style', settings: section.settings %}">
```

Reads: `content_direction`, `horizontal_alignment`, `vertical_alignment`, `horizontal_alignment_flex_direction_column`, `vertical_alignment_flex_direction_column`, `align_baseline`, `vertical_on_mobile`, `gap`.

### `gap-style` — Gap Variable

Outputs a single `--gap` CSS variable with optional responsive scaling.

```liquid
{% render 'gap-style', value: 12 %}
{% render 'gap-style', value: block.settings.gap, name: 'custom-gap' %}
```

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `value` | number | — | Gap in pixels |
| `name` | string | `gap` | CSS variable name |
| `scale_min` | number | 24 | Threshold for responsive scaling |
| `disable_scaling` | boolean | false | Disable responsive scaling |

### `size-style` — Width/Height

Outputs CSS custom properties for element sizing.

```liquid
<div class="size-style" style="{% render 'size-style', settings: block.settings %}">
```

Reads: `width`, `custom_width`, `width_mobile`, `custom_width_mobile`, `height`, `custom_height`.

### `border-override` — Border Styling

Outputs CSS custom properties for borders.

```liquid
<div style="{% render 'border-override', settings: section.settings %}">
```

Reads: `border`, `border_width`, `border_opacity`, `border_radius`.

### `background-media` — Background Image/Video

Renders a background image or video element.

```liquid
{% render 'background-media',
  background_media: section.settings.background_media,
  background_video: section.settings.video,
  background_video_position: section.settings.video_position,
  background_image: section.settings.background_image,
  background_image_position: section.settings.background_image_position
%}
```

### `overlay` — Overlay Layer

Renders a solid or gradient overlay.

```liquid
{% render 'overlay', settings: section.settings, layer: '0' %}
```

Reads: `overlay_color`, `overlay_style`, `gradient_direction`.

### `text` — Text Block

Renders a text block with typography settings.

```liquid
{% render 'text', block: block %}
```

### `button` — Button

Renders a styled button/link.

```liquid
{% render 'button',
  link: block.settings.link,
  label: block.settings.label,
  style_class: block.settings.style_class,
  open_in_new_tab: block.settings.open_in_new_tab
%}
```

---

## CSS Utility Classes

| Class | Purpose |
|-------|---------|
| `section--page-width` | Constrains content to page width |
| `section--full-width` | Full viewport width |
| `layout-panel-flex` | Base flex container |
| `layout-panel-flex--row` | Horizontal flex direction |
| `layout-panel-flex--column` | Vertical flex direction |
| `mobile-column` | Stack horizontally on mobile |
| `spacing-style` | Applies padding CSS variables |
| `gap-style` | Applies gap CSS variable |
| `size-style` | Applies width/height CSS variables |
| `border-style` | Applies border CSS variables |
| `color-{scheme-id}` | Applies a color scheme (e.g. `color-scheme-1`) |
| `page-width` | Alias for page-width constraint |
| `h1` ... `h6` | Typography preset classes |
| `paragraph` | Body text preset |

---

## JavaScript Patterns

### Component Base Class

All custom elements extend `Component` (from `assets/component.js`), which provides:

- `refs` — query elements with `ref` attributes
- `requiredRefs` — refs that must exist for the component to function
- Mutation observers for dynamic ref updates
- Event listener management via `data-*` attributes

### Import Map

Modules are registered in `snippets/scripts.liquid` via an import map:

```js
{
  "imports": {
    "@theme/component": "/assets/component.js",
    "@theme/dialog": "/assets/dialog.js",
    "@theme/events": "/assets/events.js",
    "@theme/utilities": "/assets/utilities.js",
    "@theme/section-renderer": "/assets/section-renderer.js",
    ...
  }
}
```

### Section Renderer

For sections that need dynamic content updates (e.g. filtering, pagination):

```js
import { SectionRenderer } from '@theme/section-renderer';

const renderer = new SectionRenderer(sectionId);
await renderer.renderSection(sectionId, { cache: true });
```

### Key Modules

| Module | Purpose |
|--------|---------|
| `@theme/component` | Base custom element class |
| `@theme/dialog` | Modal/dialog management |
| `@theme/events` | Custom event utilities |
| `@theme/utilities` | General helpers |
| `@theme/section-renderer` | Section Rendering API client |
| `@theme/morph` | DOM morphing for updates |
| `@theme/focus` | Focus management |
| `@theme/scrolling` | Scroll utilities |

---

## Quick Reference: Block Types Available

### Standard Theme Blocks (`@theme`)

These can be added to any section that allows `@theme` blocks:

| Block | File | Purpose |
|-------|------|---------|
| `text` | `blocks/text.liquid` | Rich text / headings |
| `button` | `blocks/button.liquid` | CTA button/link |
| `image` | `blocks/image.liquid` | Single image |
| `icon` | `blocks/icon.liquid` | SVG icon |
| `group` | `blocks/group.liquid` | Container for nested blocks |
| `spacer` | `blocks/spacer.liquid` | Vertical/horizontal space |
| `accordion` | `blocks/accordion.liquid` | Collapsible FAQ-style |
| `video` | `blocks/video.liquid` | Video player |
| `logo` | `blocks/logo.liquid` | Store logo |
| `email-signup` | `blocks/email-signup.liquid` | Newsletter form |
| `contact-form` | `blocks/contact-form.liquid` | Contact form |
| `collection-card` | `blocks/collection-card.liquid` | Collection card |
| `product-card` | `blocks/product-card.liquid` | Product card |
| `comparison-slider` | `blocks/comparison-slider.liquid` | Before/after slider |
| `custom-liquid` | `blocks/custom-liquid.liquid` | Raw Liquid code |

### Internal Blocks (Prefix `_`)

Used by sections via `{% content_for 'block', type: '_name' %}`:

| Block | Purpose |
|-------|---------|
| `_content` | Content area wrapper |
| `_media` | Media element |
| `_heading` | Heading text |
| `_card` | Card layout |
| `_divider` | Section divider |
| `_marquee` | Scrolling text |
| `_carousel-content` | Carousel slide |

---

## Translation Keys

Use translation keys from `locales/en.default.schema.json` for all setting labels. Common prefixes:

| Prefix | Example | Purpose |
|--------|---------|---------|
| `t:names.*` | `t:names.hero` | Section/block names |
| `t:settings.*` | `t:settings.color_scheme` | Setting labels |
| `t:options.*` | `t:options.center` | Select option labels |
| `t:content.*` | `t:content.layout` | Setting group headers |
| `t:categories.*` | `t:categories.text` | Preset categories |
| `t:text_defaults.*` | `t:text_defaults.shop_now_button_label` | Default text values |
| `t:html_defaults.*` | `t:html_defaults.new_arrivals_h1` | Default rich text HTML |
