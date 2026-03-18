# 00 — Global Navigation, Master Layout & Conventions

## 1. Introduction

This document defines the visual and navigation standards shared by all screens of the NMS (Network Monitoring System) for IEC 61850 substations. The target audience is the **UI/UX designer** responsible for creating mockups in Figma.

All screen wireframes (documents `01` to `09`) reference this document as the source of truth for:

- Master layout structure (header, sidebar, content area)
- Reusable components (tables, drawers, filters, pagination, empty states)
- ASCII symbol legend used in wireframes
- RBAC rules (role-based permissions)
- Timestamp conventions and formatting

---

## 2. Master Layout

The layout is composed of three fixed regions — **Header**, **Sidebar** and **Content Area** — present on all authenticated screens. The only exception is **Screen 8 (Authentication)**, which renders only the centered content area without header or sidebar.

```
+-------------------------------------------------------------------------+
| HEADER                                                                  |
| [GE Vernova Logo]  Network Monitoring System  Sub: {name}  [* RUNNING]  |
+----------------+--------------------------------------------------------+
| SIDEBAR        |                                                        |
|                |                    CONTENT AREA                        |
| 1. Dashboard   |                   (varies per screen)                  |
| 2. Protocols   |                                                        |
|    2.1 GOOSE   |                                                        |
|    2.2 SV      |            +------------------------------+            |
|    2.3 MMS     |            |    DRAWER (side panel)       |            |
|    2.4 PTP     |            |    (opens from the right)    |            |
|    2.5 SNMP    |            +------------------------------+            |
| 3. Alerts      |                                                        |
| 4. Devices     |                                                        |
| 5. Topology    |                                                        |
| 6. Redundancy  |                                                        |
| 7. Analytics   |                                                        |
| 8. Log         |                                                        |
| 9. Settings    |                                                        |
|    9.1 SCD     |                                                        |
|    9.2 Monitor |                                                        |
|    9.3 MIB     |                                                        |
|                |                                                        |
| [User Area]    |                                                        |
+----------------+--------------------------------------------------------+
```

**Notes:**

- The header spans the full width at the top of the screen.
- The sidebar is fixed on the left. It supports two states: **expanded** (255 px, icons + labels) and **collapsed** (~48 px, icons only). It has its own scroll when needed.
- The content area fills the remaining space and has independent scroll.
- The drawer, when open, partially overlays the content area from the right.

---

## 3. Header

The header is a fixed horizontal bar at the top, present on all authenticated screens.

### Components (left to right)

| Component | Position | Description | Data Source |
|---|---|---|---|
| **GE Vernova Logo** | Left | Brand logo (dark/light variants for theme). Click returns to Dashboard (`/`). | Static |
| **Application Title** | Center | Displays "Network Monitoring System". | Static |
| **Active Substation** | Center-right | Name of the substation being monitored. Shows the `substationName` field from the active SCD. | `GET /api/v1/scds?latest=true` → get `scdId` → `GET /api/v1/scds/{scdId}/summary` → `substationName` |
| **Monitoring Status** | Right | Visual badge showing the current monitoring engine state. | `GET /api/v1/monitoring` → `state` |

### Monitoring States (`state` field)

| State | Visual Indicator | Color |
|---|---|---|
| `RUNNING` | `* RUNNING` | Green |
| `STARTING` | `* STARTING` | Green (pulsing) |
| `STOPPED` | `o STOPPED` | Gray |
| `STOPPING` | `o STOPPING` | Gray (pulsing) |
| `ERROR` | `! ERROR` | Red |

### Alert Banner

When there are unacknowledged alarms (`alertCount > 0`), a conditional **amber banner** is displayed below the header bar spanning full width.

```
+-----------------------------------------------------------------------+
| ! {count} alerts waiting for acknowledgement       [Go to Alerts]     |
+-----------------------------------------------------------------------+
```

- The banner includes the alarm count and a button linking to `/alarms`.
- The banner is hidden when there are no unacknowledged alarms.

---

## 4. Sidebar (Navigation Menu)

The sidebar is a fixed vertical panel on the left, present on all authenticated screens. On **Screen 8 (Authentication)** the sidebar is not displayed.

### Menu Items

| # | Item | Icon (Phosphor) | Route | Note |
|---|---|---|---|---|
| 1 | Dashboard | SquaresFour | `/` | Landing page after login. |
| 2 | Protocols | Lightning | submenu | Expandable submenu. |
| 2.1 | → GOOSE | — | `/goose` | — |
| 2.2 | → Sampled Values | — | `/sampled-values` | — |
| 2.3 | → MMS | — | `/mms` | — |
| 2.4 | → PTP | — | `/ptp` | — |
| 2.5 | → SNMP | — | `/snmp` | — |
| 3 | Alerts | Warning | `/alarms` | Badge with unacknowledged alarm count: `GET /api/v1/alarms?ack=false` → `meta.total`. |
| 4 | Devices | Network | `/connections` | IED connections view. |
| 5 | Topology | BoundingBox | `/topology` | Substation topology graph. |
| 6 | Redundancy | ShieldCheck | `/redundancy` | HSR/PRP redundancy diagnostics derived from GOOSE/SV parser data. |
| 7 | Analytics | ChartBar | `/analytics` | — |
| 8 | Log | ClockCounterClockwise | `/log` | — |
| 9 | Settings | GearSix | submenu | Expandable submenu. |
| 9.1 | → SCD | — | `/settings/scd` | SCD upload and management. |
| 9.2 | → Monitoring | — | `/settings/monitoring` | Start/stop monitoring engine. |
| 9.3 | → MIB | — | `/settings/mib` | MIB management. |
| — | User Area | — | — | Footer of the sidebar, separated visually from nav items. |

### Sidebar States

The sidebar supports two display states, toggled by the user:

| State | Width | Content | Submenus |
|---|---|---|---|
| **Expanded** | 255 px | Icons + labels | Visible when parent is open. Children shown with left border indent. Caret indicator: `>` (collapsed) / `v` (expanded). |
| **Collapsed** | ~48 px | Icons only (32×32 touch targets) | Not displayed. Parent icons toggle internal open state; submenu becomes visible when sidebar is expanded again. |

### Submenu Behavior

- Click on a parent item (Protocols, Settings) to **expand or collapse** its children.
- Submenus are only visible in **expanded** sidebar mode.
- Children are displayed with a **left border indent** below the parent item.
- If a child route is active on page load, the parent submenu **auto-expands**.

### Active Item Indication

- The active item (current screen) has a clear visual highlight (colored background with primary color).
- For **direct links**: active when `pathname === route`.
- For **parent items**: highlighted when any child route matches the current pathname.

### User Area

The user area is positioned at the **bottom of the sidebar**, separated from navigation items by a top border.

**Expanded state:**

```
+------------------------------------------+
| (avatar) Admin User            [...]     |
|          ADMIN                           |
+------------------------------------------+
```

**Collapsed state:**

```
+--------+
| [...]  |
|  (av)  |
+--------+
```

**Components:**

- **Avatar circle** — displays a user icon with primary color background.
- **User name** — `displayName` from session/JWT claims.
- **User role** — displayed below the name in muted text.
- **Three-dot menu** `[⋮]` — opens a dropdown with:
  - **Preferences** — navigates to `/preferences`.
  - **Logout** — triggers the logout confirmation dialog.

### Logout Flow

The logout option is accessed via the **user area dropdown menu** (not a standalone sidebar item).

1. User clicks "Logout" in the dropdown menu.
2. A **confirmation dialog** appears (not a drawer):
   - Title: "Logout"
   - Description: "Are you sure you want to logout?"
   - Actions: [Cancel] [Logout]
3. On confirmation: `POST /api/auth/logout` → redirect to `/login`.

---

## 5. ASCII Wireframe Conventions

All wireframe documents (`01` to `09`) use the following symbol legend:

```
+----------------------------------------------+   Block/container with border
|  Content                                     |
+----------------------------------------------+

[Button]                Clickable button
[Button (disabled)]     Button disabled by role/state

< Navigation link >     Link to another screen

(...expandable...)      Collapsible/expandable area

> Action                Interaction flow (click result)

*  Active state         Status indicator (green)
o  Inactive state       Status indicator (gray)
!  Alert/warning        Status indicator (yellow)
X  Error/close          Error indicator (red) / Close button [X]

{Filter v}              Dropdown/select
[____]                  Text input field
(*) / o                 Radio button (selected / not)
[x] / [ ]               Checkbox (checked / not)

< 1 2 3 ... N >         Pagination
```

---

## 6. Common Patterns (Reusable Components)

The components below are referenced by multiple screens. The designer should create reusable Figma components for each of them.

---

### 6.1 Pagination

All API listings are paginated with the following parameters:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | `1` | Page number (1-based) |
| `pageSize` | integer | `50` | Items per page (maximum 200) |
| `sort` | string | varies | Sort field |
| `order` | string | `desc` | Direction: `asc` or `desc` |

The API response includes a `meta` object with the following structure:

```json
{
  "meta": {
    "page": 1,
    "pageSize": 50,
    "total": 237
  }
}
```

**Pagination wireframe:**

```
< Previous  Page 1 of 5  Next >   Showing 50 of 237
```

- The "Previous" button is disabled on the first page.
- The "Next" button is disabled on the last page.
- Total pages calculated on the frontend: `Math.ceil(total / pageSize)`.
- The "Showing X of Y" count indicates how many items are visible on the current page versus the total.

---

### 6.2 Drawer (Side Panel)

The drawer is the primary pattern for displaying details. **No modals/pop-ups are used in any screen** — all detail views use the drawer.

**Characteristics:**

- Opens from the **right**, partially overlaying the content area.
- Width: approximately **40% of the content area**.
- Close button `[X]` at the top-right corner of the drawer.
- Content behind the drawer may be slightly dimmed (optional overlay).
- The drawer has its own scroll for long content.

**Drawer wireframe:**

```
                    +---------------------------+
                    | [X]                       |
                    | Item Title                |
                    | ---------------------     |
                    | Field: Value              |
                    | Field: Value              |
                    |                           |
                    | [Primary Action]          |
                    +---------------------------+
```

---

### 6.3 Data Table

Standard pattern for all tabular listings in the system.

**Characteristics:**

- Columns support **sorting** (indicated by arrows `▲`/`▼`).
- Clicking a row opens the drawer with item details.
- Pagination positioned below the table (per section 6.1).
- Rows may have visual highlighting by severity or state (e.g., critical alarms in red).

**Table wireframe:**

```
+-----------+------------+----------+---------+
| Column ^  | Column     | Column v | Column  |
+-----------+------------+----------+---------+
| data      | data       | data     | data    |
| data      | data       | data     | data    |
+-----------+------------+----------+---------+
< Previous  Page 1 of N  Next >
```

---

### 6.4 Filter Bar

Positioned **above the data table**, allows the user to refine displayed results.

**Characteristics:**

- Filters are applied as query parameters in the API call.
- The "Clear" button resets all filters to their default state.
- Date filters use `From` / `To` fields (mapped to `fromUtc` / `toUtc` in the API).

**Filter bar wireframe:**

```
+-----------------------------------------------------+
| {Filter 1 v}  {Filter 2 v}  [From: ____] [To: ____] |
|                                          [Clear]    |
+-----------------------------------------------------+
```

---

### 6.5 Empty State

Displayed when a listing has no items to show (filter with no results, no data registered, etc.).

**Characteristics:**

- Centered in the content area.
- Illustrative icon related to the context (e.g., bell for alarms, gear for settings).
- Descriptive message explaining the empty state reason.
- Suggested action link when applicable (e.g., "Upload an SCD file").

**Empty state wireframe:**

```
+---------------------------------------------+
|                                             |
|          (illustrative icon)                |
|                                             |
|     Descriptive empty state message         |
|     < Suggested action link >               |
|                                             |
+---------------------------------------------+
```

---

### 6.6 Error Feedback

Error messages are displayed **inline** (never in a modal). The pattern is a colored banner at the top of the content area.

**Characteristics:**

- Alert icon on the left.
- Descriptive error message.
- Technical details when applicable (error code, endpoint, etc.).
- Close button `[X]` on the right.
- Colors: red for errors, yellow for warnings.

**Error feedback wireframe:**

```
+-------------------------------------------------+
| ! Descriptive error message                     |
| Technical details (when applicable)       [X]   |
+-------------------------------------------------+
```

---

### 6.7 Dynamic Fields (Key-Value)

Some API fields have `additionalProperties: true`, meaning the key structure is not fixed. Examples:

- `Alarm.details` — varies by alarm type
- `MetricPoint.metrics` — key-value pairs that differ by protocol
- `Scd.parsed` — full parsed SCD content
- `ScdProtocolDetailsResponse.data` — protocol-specific

These fields should be rendered as a **dynamic key-value table**, without hardcoded components for specific keys.

**Dynamic fields wireframe:**

```
+------------------+--------------------+
| Key              | Value              |
+------------------+--------------------+
| field1           | value1             |
| field2           | value2             |
+------------------+--------------------+
```

---

## 7. Role-Based Permissions (RBAC)

The system has three access roles, defined by the `UserRole` API enum:

| Role | Description | Allowed Actions |
|---|---|---|
| **ADMIN** | System administrator | All actions: configuration (SCD upload, MIB management), monitoring control (start/stop), alarm acknowledgement (ACK), viewing all screens. |
| **OPERATOR** | Substation operator | Monitoring control (start/stop), alarm acknowledgement (ACK), viewing all screens. **No access** to SCD or MIB configuration. |
| **VIEWER** | Read-only viewer | View only. No write actions (no start/stop, no ACK, no configuration). |

### General Display Rule

- Buttons and actions **unavailable** to the user's role should be **hidden** (not displayed).
- **Exception**: when the user needs to know the action exists but lacks permission, the button should be displayed as **disabled** with an explanatory tooltip (e.g., "Only administrators can perform this action").
- The decision to hide or disable should be evaluated case by case in each screen wireframe.

### Permission Summary by Feature

| Feature | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| View all screens | Yes | Yes | Yes |
| Start/Stop monitoring | Yes | Yes | No |
| ACK alarms | Yes | Yes | No |
| Upload SCD | Yes | No | No |
| Confirm/Reject SCD | Yes | No | No |
| Manage MIBs | Yes | No | No |
| Download PCAPs | Yes | Yes | Yes |

---

## 8. Timestamps

### API Convention

- All timestamps returned by the API are in **UTC** in **ISO 8601** format (e.g., `2026-02-19T14:30:00Z`).
- Fields follow the `*Utc` naming convention (e.g., `createdAtUtc`, `updatedAtUtc`, `timestampUtc`, `parsedAtUtc`).

### Frontend Display

- The frontend is responsible for converting UTC timestamps to the **user's local timezone** before display.
- Suggested display format: `DD/MM/YYYY HH:mm:ss`
- Examples:
  - API returns: `2026-02-19T14:30:00Z`
  - Display (BRT, UTC-3): `19/02/2026 11:30:00`

### Recommendation for the Designer

- Allow sufficient space for the `DD/MM/YYYY HH:mm:ss` format (19 characters) in all date/time table columns.
- In space-constrained contexts (badges, cards), consider abbreviated format: `DD/MM HH:mm`.

---

## 9. Document Reference

This is the complete wireframe index for NMS. Each document details a specific screen and references this document (`00`) for shared patterns.

| File | Screen | Route | Description |
|---|---|---|---|
| `09-dashboard.md` | Dashboard | `/` | **NEW** — Substation overview. Landing page after login. |
| `01-tela-inicial.md` | Topology | `/topology` | Substation topology with IEDs, networks, and GOOSE/SV flows. No longer the landing page. |
| `02-alarmes.md` | Alarms | `/alarms` | Alarm listing, filters, details, and acknowledgement (ACK). |
| `03-sincronismo-temporal.md` | PTP Synchronization | `/ptp` | PTP (Precision Time Protocol) synchronization monitoring of IEDs. |
| `04-redundancia-rede.md` | Redundancy (HSR/PRP) | `/redundancy` | HSR/PRP network redundancy monitoring. |
| `05-comunicacao-dados.md` | GOOSE / SV / MMS | `/goose`, `/sampled-values`, `/mms` | Protocol communication monitoring. Individual routes per protocol. |
| `06-snmp.md` | SNMP | `/snmp` | SNMP monitoring of switches and network equipment. |
| `07-configuracao.md` | Settings | `/settings/scd`, `/settings/monitoring`, `/settings/mib` | SCD management, monitoring control, and MIB management. Sub-routes per tab. |
| `08-autenticacao.md` | Authentication | `/login` | Login screen. Only screen without header and sidebar. |
| — | Connections | `/connections` | Wireframe pending. |
| — | Analytics | `/analytics` | Wireframe pending. |
| — | Log | `/log` | Wireframe pending. |
