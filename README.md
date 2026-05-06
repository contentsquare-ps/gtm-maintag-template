# Contentsquare Main Tag — Google Tag Manager Template

A GTM **Custom Tag Template** for deploying the Contentsquare analytics script on web containers. Handles initial page-load injection, Single-Page Application (SPA) pageviews, WebView support, UTM/GCLID tracking, Custom Variables, and PII masking — all from a single tag configuration.

---

## Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
  - [Tag IDs](#tag-ids)
  - [Custom Variables](#custom-variables)
  - [PII Masking](#pii-masking)
- [How It Works](#how-it-works)
- [Permissions](#permissions)
- [Changelog](#changelog)

---

## Overview

| Field | Value |
|---|---|
| **Template type** | Tag |
| **Container context** | Web |
| **Categories** | Analytics, Heat Map, Session Recording |
| **Homepage** | https://contentsquare.com/ |
| **Documentation** | https://docs.contentsquare.com/uxa-en/ |
| **Developer** | Uri Gobey @ Contentsquare |

---

## Requirements

- A Google Tag Manager **Web** container.
- A Contentsquare **Main Tag ID** (and optionally a **WebView Tag ID**).
- The template imported into your GTM workspace via **Templates → Search Gallery** or by importing the `.tpl` file directly.

---

## Installation

1. In GTM, go to **Templates → Tag Templates → New**.
2. Click the overflow menu (⋮) and choose **Import**.
3. Upload `template.tpl` from this repository.
4. Save the template.
5. Create a new tag, select **Contentsquare - Main tag**, and configure the fields below.

---

## Configuration

### Tag IDs

| Field | Required | Description |
|---|---|---|
| **Main Tag ID** | Conditional | Your unique Contentsquare tag identifier (e.g. `123456789`). Required unless you are deploying a WebView-only setup. |
| **WebView Tag ID** | Optional | A separate tag ID used when the page is rendered inside a WebView. Leave blank if not applicable. |

> **GUIDs are supported** in both ID fields.

### Custom Variables

Up to 20 custom variables can be configured in the **Contentsquare Custom Variables** table. Each row defines:

| Column | Description |
|---|---|
| **Slot** | Integer 1–20. Each slot maps to a Contentsquare custom variable slot. Must be unique. |
| **Name** | The variable name sent to Contentsquare. Must be non-empty and unique. |
| **Value** | The variable value. Supports GTM variable references (e.g. `{{Page Path}}`). |
| **Scope** | Controls when the variable is associated with a session or page (see table below). |

#### Scope values

| Scope label | Numeric value | Behaviour |
|---|---|---|
| Page (default) | `3` | Sent on every page view. |
| Visit | `2` | Sent once per visit/session. |
| Page & Visit | `2 & 3` | Sent with both scopes simultaneously. |
| Single Page App (nextPageOnly) | `4` | Sent only on the next SPA virtual pageview. |

> Tags imported from older versions of this template that had no scope field default to **Page (3)** for backward compatibility.

### PII Masking

Sensitive content can be excluded from Contentsquare session replays.

#### CSS Selectors (text nodes)

Provide a comma-separated list of CSS selectors whose **text content** will be masked.

```
#username, .user-email, #firstname
```

#### Attribute & Selector pairs (element data attributes)

Add rows to the table to mask specific **data attributes** on matching elements.

| Column | Example | Description |
|---|---|---|
| CSS Selector | `#submit-button, .formfield` | Targets the element(s). |
| Data Attributes | `data-email, data-firstname` | Comma-separated data attributes to mask on the matched element(s). |

---

## How It Works

The tag uses the `_uxa` command queue (Contentsquare's push API) and follows this execution flow on every GTM trigger:

```
Tag fires
│
├── Push UTM parameters as dynamic variables (utm_medium, utm_source, utm_campaign)
├── Push gclid presence as a dynamic variable
├── Push all configured Custom Variables (setCustomVariable)
├── Push PII masking rules if configured (setPIISelectors)
│
└── Is window.CS_CONF defined?
    ├── YES → SPA subsequent pageview path
    │         _uxa.push(["trackPageview", pathname + hash])
    │         ✓ gtmOnSuccess
    │
    └── NO  → Initial page load path
              setPath (pathname + hash, with "#" normalised to "?__")
              │
              └── Is WebView mode active? (CS_webViewEnabled or CS_isWebView)
                  ├── YES → inject WebView Tag ID script
                  └── NO  → inject Main Tag ID script
                            https://t.contentsquare.net/uxa/<TagID>.js
                            ✓ gtmOnSuccess / ✗ gtmOnFailure
```

**Hash handling:** URL fragments are normalised from `#section` to `?__section` so Contentsquare can track hash-based navigation as distinct page paths.

**SPA support:** When `CS_CONF` is already present on the window (i.e., the Contentsquare script was loaded on a previous virtual page), the tag skips re-injection and calls `trackPageview` directly instead, preventing duplicate script loads.

---

## Permissions

The template declares the following GTM sandbox permissions:

| Permission | Details |
|---|---|
| `access_globals` | Read/write `_uxa` (command queue); read/write `CS_CONF`; read `CS_webViewEnabled`; read `CS_isWebView` |
| `inject_script` | Scripts from `https://t.contentsquare.net/*` |
| `get_url` | Full URL including pathname, hash, and any query parameters |

---

## Changelog

| Change notes |
|---|
| Fixed bug in APV |
| Added support for WebView tag |
| Refactored Scope 4 logic |
| Fixed bug with table (GTM bug) |
| Removed setQuery; UI improvements; improved Scope 4 custom vars for artificial pageview |
| Fixed bug with setQuery |
| Added support for more custom variable scenarios |
| Re-added setQuery |
| Updated company logo |
| Added Scope setting for Custom Variables |
| Updated brand name |
| Added missing `gtmOnSuccess` callback in else branch (fixes "Still Running" in GTM debugger for SPA) |
| Added PII masking feature |
| Added `setPath` functionality |
| Fixed disappeared custom variables dropdown |
| Fixed hash collection |
| Added support for GUIDs in the Tag ID field |
| First version |

