# Branding & Design System

## Brand Identity

**Product Name:** AegisMCP
**Tagline:** The Zero-Trust Security Firewall and Governance Gateway for Model Context Protocol Agents
**Brand Personality:** Authoritative, uncompromising, cryptographic, high-performance, calm
**Tone of Voice:** Precise, technical, reassuring, and security-first with zero decorative fluff

---

## Design Character

**Audience:** Platform Engineers, DevSecOps Leads, and Enterprise Compliance Officers (see [user-personas](user-personas.md)).

**Feels like:** Cryptographically secure, razor-sharp, terminal-clean, telemetry-rich, and dependable.

**Must not feel like:** Toy-like, playful, marketing-heavy, opaque, or like a consumer conversational chatbot.

**Motion character:** Instant and mechanical — micro-interactions feel like precision-milled hardware switches, with status changes and streaming telemetry updating with zero sluggishness.

**Research basis:**
- [Tailscale](https://tailscale.com) — borrowed the minimalist, developer-first security posture: high information density, clear tabular layouts, and clear device/access state indicators.
- [Cloudflare Dashboard](https://dash.cloudflare.com) — borrowed the real-time firewall analytics mental model: high-contrast security event flags, clear allow/block visual dichotomy, and rapid rule-filtering ergonomics.

---

## Brand Mark & Logo System

**Mark type:** Combination mark (fused geometric symbol + technical letterform wordmark)

**Category shorthand:** A medieval heater shield or an oversized metallic padlock.

**Concept:** A circuit trace mesh whose junction node becomes a faceted protective bastion.

**Direction:** A sharp, geometric hexagonal node anchored by converging data flow lines: incoming protocol vectors meet at a central hexagonal bastion that deflects unauthorized paths while channeling verified traffic through a clean axial conduit. Technical, monolithic, solid.

**Intended reading:** Deterministic perimeter security — active traffic verification and immutable defense.

**Failure reading:** An ambiguous cryptocurrency coin or a generic antivirus badge.

**Type treatment:** "Aegis" is rendered in a high-contrast geometric sans-serif with tight tracking and a sharp vertex; "MCP" is set in a precision monospace face (`JetBrains Mono`) enclosed within a subtle outlined badge container.

**Usage rules:** The hexagonal node stands alone as the favicon, browser app icon, and CLI terminal badge; in horizontal console headers, the mark leads the wordmark with an exact 12px gap. The mark is never placed on busy photo backgrounds or styled with multi-color novelty gradients.

**Required forms:** Mark-only (app icon / favicon / CLI badge), horizontal lockup (web console header), wordmark-only, one-colour monochrome (terminal output), knockout on dark.

**Surfaces:** Dark terminal backgrounds (`#0B0F19`), enterprise light surface (`#F8FAFC`), and cloud console canvas.

**One tone & reversal:** Solid crisp white or deep slate; reverses cleanly on dark as the primary ink token.

**Scale & recognition:** Legible at 16x16px as a clean closed silhouette without losing interior structure.

---

## Design System

### Responsive & Device Targets

| Device class | Width range | Input modality | Layout posture |
|---|---|---|---|
| Mobile | < 600px | Touch | Single column, stacked approval cards |
| Tablet | 600-1024px | Touch + pointer | Adaptive columns, collapsible navigation rail |
| Desktop | > 1024px | Pointer + keyboard | Multi-column, persistent sidebar, live traffic stream inspector |

**Base:** Mobile-first — mobile values form the base; larger viewport classes expand and adapt.
**Breakpoints:** 600px, 1024px, 1440px.

---

### Color Palette

#### Primary Colors — a 9-shade scale
| Token | Light | Dark | Role |
|---|---|---|---|
| `primary-900` | `#0A192F` | `#93C5FD` | Darkest — primary brand text on light surfaces |
| `primary-800` | `#172554` | `#60A5FA` | Deep brand emphasis |
| `primary-700` | `#1E3A8A` | `#3B82F6` | Pressed states, active navigation |
| `primary-600` | `#1D4ED8` | `#2563EB` | Hover states on primary controls |
| `primary-500` | `#2563EB` | `#3B82F6` | **Base** — primary buttons, active links, focus rings |
| `primary-400` | `#60A5FA` | `#1D4ED8` | Secondary borders, tinted highlights |
| `primary-300` | `#93C5FD` | `#1E3A8A` | Selected row borders, active tab indicator |
| `primary-200` | `#BFDBFE` | `#172554` | Light badge background |
| `primary-100` | `#EFF6FF` | `#0A192F` | Lightest — tinted notification and row backgrounds |

#### Neutral Colors — a 9-shade slate scale
| Token | Light | Dark | Role |
|---|---|---|---|
| `grey-900` | `#0F172A` | `#F8FAFC` | Darkest — primary text |
| `grey-800` | `#1E293B` | `#F1F5F9` | Heavy headings |
| `grey-700` | `#334155` | `#E2E8F0` | High-contrast secondary text |
| `grey-600` | `#475569` | `#CBD5E1` | Standard secondary text |
| `grey-500` | `#64748B` | `#94A3B8` | Muted labels, metadata, placeholders |
| `grey-400` | `#94A3B8` | `#64748B` | Disabled icons, subtle borders |
| `grey-300` | `#CBD5E1` | `#334155` | Card dividers, table borders |
| `grey-200` | `#E2E8F0` | `#1E293B` | Input borders, secondary button background |
| `grey-100` | `#F1F5F9` | `#0F172A` | Secondary surfaces, code block backgrounds |
| `grey-50` | `#F8FAFC` | `#0B0F19` | Root page background |

#### Accent & Semantic Colors — 5-shade scales
| Token | Light | Dark | Role |
|---|---|---|---|
| `success-900` | `#064E3B` | `#A7F3D0` | Success text on light surfaces |
| `success-700` | `#047857` | `#6EE7B7` | Success borders and active icons |
| `success-500` | `#10B981` | `#34D399` | Allowed tool calls, verified policy passes |
| `success-300` | `#6EE7B7` | `#047857` | Success pill borders |
| `success-100` | `#ECFDF5` | `#064E3B` | Allowed tool call badge backgrounds |
| `warning-900` | `#78350F` | `#FDE68A` | Warning text on light surfaces |
| `warning-700` | `#B45309` | `#FCD34D` | Warning borders and active icons |
| `warning-500` | `#F59E0B` | `#FBBF24` | Human-in-the-Loop approval pending, rate warnings |
| `warning-300` | `#FCD34D` | `#B45309` | Warning pill borders |
| `warning-100` | `#FFFBEB` | `#78350F` | Pending approval badge backgrounds |
| `error-900` | `#7F1D1D` | `#FECACA` | Error text on light surfaces |
| `error-700` | `#B91C1C` | `#F87171` | Destructive action hover, critical alerts |
| `error-500` | `#EF4444` | `#F87171` | Blocked tool calls, prompt injection detections |
| `error-300` | `#FCA5A5` | `#B91C1C` | Error pill borders |
| `error-100` | `#FEF2F2` | `#7F1D1D` | Blocked violation row backgrounds |
| `info-900` | `#164E63` | `#A5F3FC` | Info text on light surfaces |
| `info-700` | `#0E7490` | `#67E8F9` | Informational borders and icons |
| `info-500` | `#06B6D4` | `#22D3EE` | Telemetry highlights, JSON-RPC protocol notices |
| `info-300` | `#67E8F9` | `#0E7490` | Info pill borders |
| `info-100` | `#ECFEFF` | `#164E63` | Informational callout backgrounds |
| `accent-900` | `#4C1D95` | `#DDD6FE` | Agent identity badge text |
| `accent-700` | `#6D28D9` | `#C4B5FD` | Agent role highlight borders |
| `accent-500` | `#8B5CF6` | `#A78BFA` | Autonomous agent identities, SPIFFE badges |
| `accent-300` | `#C4B5FD` | `#6D28D9` | Agent identity pill borders |
| `accent-100` | `#F5F3FF` | `#4C1D95` | Agent identity tag backgrounds |

---

### Typography

#### Font Family
| Role | Font | Fallback |
|---|---|---|
| Display / Headings | `Inter` | `sans-serif` |
| Body | `Inter` | `sans-serif` |
| Monospace | `JetBrains Mono` | `monospace` |

#### Type Scale
| Token | Mobile | Tablet | Desktop | Weight | Line Height | Usage |
|---|---|---|---|---|---|---|
| `display` | 28px | 34px | 40px | 700 | 1.2 | Security status overview heroes |
| `h1` | 24px | 28px | 32px | 700 | 1.3 | Console view page titles |
| `h2` | 20px | 22px | 24px | 600 | 1.35 | Section headers, panel titles |
| `h3` | 18px | 19px | 20px | 600 | 1.4 | Inspection card titles, modal headings |
| `body-large` | 17px | 18px | 18px | 400 | 1.5 | Policy description leads, approval prompts |
| `body` | 16px | 16px | 16px | 400 | 1.5 | General UI text, form labels |
| `body-small` | 14px | 14px | 14px | 400 | 1.5 | Log entries, table cell content |
| `caption` | 12px | 12px | 12px | 500 | 1.4 | Status badges, timestamps, protocol tags |
| `mono-code` | 13px | 13px | 14px | 400 | 1.45 | JSON-RPC payloads, tool parameter ASTs |

---

### Spacing & Sizing

#### Spacing
| Token | Mobile | Tablet | Desktop | Usage |
|---|---|---|---|---|
| `space-xs` | 4px | 4px | 4px | Tight spacing — badge inner padding, status indicator dots |
| `space-sm` | 8px | 8px | 12px | Compact spacing — table row padding, input inner padding |
| `space-md` | 12px | 16px | 24px | Default spacing — card padding, grid gaps |
| `space-lg` | 20px | 24px | 32px | Section separation, modal inner gutters |
| `space-xl` | 28px | 32px | 48px | Page margins, telemetry block margins |
| `space-2xl` | 40px | 48px | 64px | Major view layout separation |
| `space-3xl` | 56px | 64px | 96px | Hero banner and dashboard boundary padding |

#### Sizing
##### Scales by device
| Token | Mobile | Tablet | Desktop | Usage |
|---|---|---|---|---|
| `button-height` | 44px | 40px | 36px | Action buttons and approval triggers |
| `input-height` | 48px | 44px | 40px | Search filters, policy inputs |
| `icon-sm` | 16px | 16px | 16px | Inline protocol status indicators |
| `icon-md` | 24px | 24px | 24px | Navigation rail and card action icons |
| `icon-lg` | 32px | 32px | 40px | Security alert banners and empty-state graphics |

##### Fixed
| Token | Value | Usage |
|---|---|---|
| `radius-sm` | 4px | Code tags, table badges, protocol pills |
| `radius-md` | 6px | Action buttons, inputs, telemetry cards |
| `radius-lg` | 8px | Approval dialogs, modal overlays, side sheets |
| `radius-full` | 9999px | Agent identity badges, status dots |

---

### Shadows & Elevation

| Token | Light | Dark | Usage |
|---|---|---|---|
| `shadow-none` | `none` | `none` | Base surface, table rows |
| `shadow-sm` | `0 1px 2px rgba(15,23,42,0.06)` | `0 1px 2px rgba(0,0,0,0.5)` | Telemetry cards, inspection panels |
| `shadow-md` | `0 4px 6px -1px rgba(15,23,42,0.1)` | `0 4px 8px rgba(0,0,0,0.6)` | Dropdowns, popovers, active tool inspection panels |
| `shadow-lg` | `0 10px 15px -3px rgba(15,23,42,0.12)` | `0 10px 24px rgba(0,0,0,0.75)` | Human approval dialogs, policy editor drawers |
| `shadow-xl` | `0 20px 25px -5px rgba(15,23,42,0.16)` | `0 20px 32px rgba(0,0,0,0.85)` | Emergency kill-switch overlays, full-screen modals |

---

### Component Tokens

#### Buttons
| Variant | Background | Text | Border | Radius | Height | Padding |
|---|---|---|---|---|---|---|
| Primary | `primary-500` | `#FFFFFF` | none | `radius-md` | `button-height` | `space-md space-lg` |
| Secondary | `primary-100` | `primary-700` | none | `radius-md` | `button-height` | `space-md space-lg` |
| Outline | transparent | `primary-500` | 1px `primary-500` | `radius-md` | `button-height` | `space-md space-lg` |
| Ghost | transparent | `grey-700` | none | `radius-md` | `button-height` | `space-md space-lg` |
| Destructive | `error-500` | `#FFFFFF` | none | `radius-md` | `button-height` | `space-md space-lg` |

#### Inputs
| State | Border | Background | Text |
|---|---|---|---|
| Default | `grey-300` | `grey-100` | `grey-900` |
| Focus | `primary-500` | `grey-50` | `grey-900` |
| Error | `error-500` | `grey-50` | `grey-900` |
| Disabled | `grey-200` | `grey-100` | `grey-400` |

#### Status Badges (Security States)
| State | Background | Text | Border |
|---|---|---|---|
| Allowed / Pass | `success-100` | `success-900` | 1px `success-300` |
| Pending HITL | `warning-100` | `warning-900` | 1px `warning-300` |
| Blocked / Quarantined | `error-100` | `error-900` | 1px `error-300` |
| Active Agent | `accent-100` | `accent-900` | 1px `accent-300` |

---

### Accessibility

**Target:** WCAG 2.2 Level AA

- **Contrast ratios:** All normal body text maintains >= 4.5:1 contrast against its background in both light and dark themes; large display headings maintain >= 3:1 contrast.
- **Color independence:** Security event states (Allowed, Blocked, Pending) are never communicated solely through color; every status badge pairs a color with an explicit semantic icon and text label (e.g. checkmark icon + "ALLOWED", shield-ban icon + "BLOCKED").
- **Touch targets:** Minimum touch targets of 44x44px are maintained on mobile and tablet form factors for interactive triggers and table row expansions.
- **Focus indicators:** Prominent 2px `primary-500` focus ring with a 2px offset on all keyboard-navigable elements.
- **Screen reader support:** Data tables for audit logs feature explicit ARIA roles, column headers, and live region announcements for incoming streaming JSON-RPC violations.
