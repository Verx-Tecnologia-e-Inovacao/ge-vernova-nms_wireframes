# Screen 7 -- Settings

## Objective

Manage the three pillars of NMS configuration for IEC 61850 substations via three independent sub-routes accessible from the Settings submenu in the sidebar:

1. **SCD Management** (`/settings/scd`) -- Upload, parse, confirmation/rejection, and activation of SCD/CID files. Includes enriched summary and detailed diagnostics from the parser.
2. **Monitoring Control** (`/settings/monitoring`) -- View status and start/stop the monitoring engine.
3. **MIB Management** (`/settings/mib`) -- CRUD of MIB files used by the SNMP agent.

**Before:** These were tabs within `/configuracao`.
**After:** Independent pages at `/settings/scd`, `/settings/monitoring`, `/settings/mib`, linked via the Settings submenu in the sidebar (see `00-navegacao-global.md` section 4).

The substation in this example has 42 real IEDs + 161 proxy nodes = 203 total nodes, distributed across 9 subnets, with 237 GOOSE streams and 80 SV streams.

### SCD Lifecycle

Before detailing each sub-route, it is important to understand the complete lifecycle of an SCD/CID file in the system:

```
           +----------+
           | UPLOADED | <-- POST /scds (file upload)
           +----+-----+
                |
                | automatic parser (async)
                |
           +----+------+    +----------+
           |           |    |  STALLED | <-- parser failed or timeout
           v           v    +----+-----+
       +--------+  +-------+    |
       | PARSED |  | ERROR |    |
       +---+----+  +-------+    |
           |                    |
     +-----+------+             |
     |            |             |
     v            v             v
 +--------+  +----------+  +---------+
 | ACTIVE |  | REJECTED |  | DELETED |
 +---+----+  +----------+  +---------+
     |
     | when another SCD is activated
     v
 +----------+
 | INACTIVE |
 +----------+

 Any status (except ACTIVE) --> DELETED (soft delete)
```

### Status badge color guide

| Status   | Color  | ASCII indicator | Meaning                              |
|----------|--------|-----------------|--------------------------------------|
| UPLOADED | Blue   | o UPLOADED      | File received, awaiting parse        |
| PARSED   | Amber  | o PARSED        | Parse complete, awaiting decision    |
| STALLED  | Orange | ! STALLED       | Parser failed or timed out           |
| ACTIVE   | Green  | * ACTIVE        | Currently active SCD                 |
| INACTIVE | Gray   | o INACTIVE      | Deactivated (replaced by newer SCD)  |
| REJECTED | Red    | ! REJECTED      | Rejected by administrator            |
| DELETED  | Dark   | o DELETED       | Soft-deleted                         |

---

## Wireframe

### Settings layout (shared across all sub-routes)

The Settings layout renders a tab bar below the global header. Each tab is a `<Link>` to its sub-route. The active tab has a bottom border highlight and bold text.

```
+--------------------------------------------------------------------------------------------------+
|  +- Header (see 00-navegacao-global.md) ---------------------------------------------------+     |
|  |  [Logo NMS]   SubstationXYZ   Mon: * RUNNING                   admin@empresa.com        |     |
|  +---------------------------------------------------------------------------------------  +     |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  +- Settings Tab Bar (shared layout) --------------------------------------------+     |
|   Dash.  |  |                                                                               |     |
|   Proto  |  |   [ SCD ]    [ Monitoring ]    [ MIBs ]                                       |     |
|   Alerts |  |   =======                                                                     |     |
|   Devic. |  |   (active)                                                                    |     |
|   Topol. |  +-------------------------------------------------------------------------------+     |
|   Redun. |                                                                                        |
|   Analy. |  +- Content of active sub-route -------------------------------------------------+     |
|   Log    |  |                                                                               |     |
| > Sett.  |  |               (renders according to selected tab)                             |     |
|   > SCD  |  |                                                                               |     |
|   > Mon. |  +-------------------------------------------------------------------------------+     |
|   > MIB  |                                                                                        |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

**Tab bar behavior:**

- The active tab has an underline/bottom border with the highlight color and bold text.
- Inactive tabs have normal text and no underline.
- Each tab is a Next.js `<Link>` navigating to its sub-route.
- The active tab is determined by the current pathname (`/settings/scd`, `/settings/monitoring`, `/settings/mib`).
- If a child route is active on page load, the Settings parent item in the sidebar auto-expands.

---

### /settings/scd -- SCD Management

#### Main state (with SCDs loaded)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header ---------------------------------------------------------------------------------+     |
|  |  [Logo NMS]   SubstationXYZ   Mon: * RUNNING                   admin@empresa.com        |     |
|  +----------------------------------------------------------------------------------------  +     |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                 |
| > Sett.  |  =======                                                                               |
|   > SCD  |                                                                                        |
|   > Mon. |  +- SCD Management ----------------------------------------------------------+         |
|   > MIB  |  |                                                              [Upload SCD]  |         |
|          |  |  Status: {All v}    [Search: ___________________________]    [Clear]       |         |
|          |  |                                                                            |         |
|          |  |  File Name                  | Status      | Created          | Parsed           | Activated  |
|          |  |  ---------------------------+-------------+------------------+------------------+------------|
|          |  |  SubstationXYZ_2026.scd     | * ACTIVE    | 26/03/2026 09:15 | 26/03/2026 09:16 | 26/03 09:20|
|          |  |  v (expanded)               | (green)     |                  |                  |            |
|          |  |                                                                                           |
|          |  |  +- Inline expansion: SubstationXYZ_2026.scd -----------------------------------+         |
|          |  |  | [Summary]  [Diagnostics]                                                      |         |
|          |  |  | (Summary tab active by default)                                               |         |
|          |  |  |                                                                               |         |
|          |  |  |  (see detailed expansion wireframes below)                                    |         |
|          |  |  |                                                                               |         |
|          |  |  +-------------------------------------------------------------------------------+         |
|          |  |                                                                                           |
|          |  |  ---------------------------+-------------+------------------+------------------+------------|
|          |  |  PUPC_3B_20260201.CID       | o PARSED    | 28/03/2026 14:30 | 28/03/2026 14:31 |     -      |
|          |  |  ---------------------------+-------------+------------------+------------------+------------|
|          |  |  DUCD_3T3.CID               | ! STALLED   | 25/03/2026 09:15 |        -         |     -      |
|          |  |  ---------------------------+-------------+------------------+------------------+------------|
|          |  |  SubstationXYZ_2025.scd     | o INACTIVE  | 15/02/2026 08:00 | 15/02/2026 08:02 | 15/02 08:10|
|          |  |                                                                                           |
|          |  |  < Prev   Page 1 of 1   Next >    Showing 4 of 4                                         |
|          |  +------------------------------------------------------------------------------------+      |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

**Table behavior:**

- Default sort: `createdAtUtc` descending (newest first).
- Sortable columns: File Name, Created.
- Click a row to expand/collapse the inline detail panel below that row.
- Only one row may be expanded at a time.
- Status column shows colored badge per status table.
- Null date fields display a dash (`-`).
- `[Upload SCD]` button is visible only for ADMIN role.

---

#### Inline expansion -- Summary tab

When a row is expanded, the detail panel shows two inner tabs: Summary and Diagnostics. Summary is active by default.

```
+--------------------------------------------------------------------------------------------------+
|  +- Inline expansion: SubstationXYZ_2026.scd -----------------------------------------+         |
|  |                                                                                      |         |
|  |  [Summary]  [Diagnostics]                                                            |         |
|  |  =========                                                                           |         |
|  |  (Summary tab active)                                                                |         |
|  |                                                                                      |         |
|  |  Substation: SubstationXYZ     Status: * ACTIVE                                     |         |
|  |  Parsed:     26/03/2026 09:16  Activated: 26/03/2026 09:20                          |         |
|  |  Source file: SubstationXYZ_2026.scd  (+41 CID files)                               |         |
|  |                                                                                      |         |
|  |  +- IEDs ------------------+  +- Streams ---------------+  +- Subscriptions ------+ |         |
|  |  |                        |  |                         |  |                      | |         |
|  |  |  Real          42      |  |  GOOSE        237       |  |  Valid       2,682   | |         |
|  |  |  Proxy        161      |  |  SV            80       |  |  Invalid         0   | |         |
|  |  |                        |  |                         |  |                      | |         |
|  |  |  By type:              |  |                         |  |                      | |         |
|  |  |  mu:       16          |  |                         |  |                      | |         |
|  |  |  ied:      20          |  |                         |  |                      | |         |
|  |  |  other:     6          |  |                         |  |                      | |         |
|  |  |  proxy:   161          |  |                         |  |                      | |         |
|  |  |                        |  |                         |  |                      | |         |
|  |  +------------------------+  +-------------------------+  +----------------------+ |         |
|  |                                                                                      |         |
|  |  +- MMS Points ------------+  +- Vendors & Source Files ---------------------------+ |         |
|  |  |                        |  |                                                    | |         |
|  |  |  Total       5,370     |  |  Vendors:       GE, GE Multilin                   | |         |
|  |  |  LGOS        4,880     |  |  Source files:  42                                | |         |
|  |  |  LSVS          408     |  |                                                    | |         |
|  |  |  LTMS           82     |  |                                                    | |         |
|  |  |                        |  |                                                    | |         |
|  |  +------------------------+  +----------------------------------------------------+ |         |
|  |                                                                                      |         |
|  |  +- Actions (ADMIN only, varies by status) ----------------------------------+      |         |
|  |  |  (This SCD is ACTIVE -- no actions available)                             |      |         |
|  |  +---------------------------------------------------------------------------+      |         |
|  |                                                                                      |         |
|  +--------------------------------------------------------------------------------------+         |
+--------------------------------------------------------------------------------------------------+
```

---

#### Inline expansion -- Diagnostics tab

When the Diagnostics tab is clicked, the inner content switches to the diagnostics accordion. The full diagnostics data is loaded lazily via `GET /scds/{scdId}/VALIDATIONS` only when this tab is first opened.

```
+--------------------------------------------------------------------------------------------------+
|  +- Inline expansion: SubstationXYZ_2026.scd -----------------------------------------+         |
|  |                                                                                      |         |
|  |  [Summary]  [Diagnostics]                                                            |         |
|  |             =============                                                            |         |
|  |             (Diagnostics tab active)                                                 |         |
|  |                                                                                      |         |
|  |  +- Diagnostics summary badges ------------------------------------------------+   |         |
|  |  |  [! 38 Errors]  [! 93 Warnings]  [o 3,095 Info]                             |   |         |
|  |  +----------------------------------------------------------------------------  +   |         |
|  |                                                                                      |         |
|  |  +- v Errors (38) [expanded by default] --------------------------------------+   |         |
|  |  |                                                                             |   |         |
|  |  |  Type                    | Message (truncated)                    | File           |
|  |  |  ------------------------+----------------------------------------+----------------|
|  |  |  goose_incomplete        | GOOSE found in Communication but GSE.. | PUPC_3B.CID    |
|  |  |  goose_incomplete        | GOOSE found in Communication but GSE.. | DUPC_0P3.CID   |
|  |  |  goose_config_mismatch   | Same control block has 2 conflicting.. | DUCD_3T3.CID   |
|  |  |  sv_config_mismatch      | Same control block has 2 conflicting.. | DUCD_3T3.CID   |
|  |  |  sv_incomplete           | SV found in Communication but Sampl... | DUPC_0P3.CID   |
|  |  |  (click row to open detail drawer)                                          |   |         |
|  |  |                                                                             |   |         |
|  |  |  < 1  2  3  4  5  6  7  8 >    Showing 1-5 of 38                           |   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  |  +- > Warnings (93) [collapsed -- click to expand] ---------------------------+   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  |  +- > Info (3,095) [collapsed -- click to expand] ----------------------------+   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  +--------------------------------------------------------------------------------------+         |
+--------------------------------------------------------------------------------------------------+
```

**Accordion behavior:**

- **Errors** section is expanded by default when the Diagnostics tab first opens.
- **Warnings** and **Info** sections are collapsed by default.
- Clicking a section header toggles its expanded/collapsed state.
- Each section, when expanded, shows a paginated table of diagnostic items (5 items per page).
- Clicking a row in any table opens the diagnostic detail drawer.
- The `>` prefix indicates a collapsed section; `v` prefix indicates expanded.
- Loading spinner shows while `GET /scds/{scdId}/VALIDATIONS` is in flight.

---

#### Diagnostics accordion -- Warnings expanded

```
+--------------------------------------------------------------------------------------------------+
|  +- Inline expansion: SubstationXYZ_2026.scd -----------------------------------------+         |
|  |                                                                                      |         |
|  |  [Summary]  [Diagnostics]                                                            |         |
|  |             =============                                                            |         |
|  |                                                                                      |         |
|  |  [! 38 Errors]  [! 93 Warnings]  [o 3,095 Info]                                     |         |
|  |                                                                                      |         |
|  |  +- > Errors (38) [collapsed] -------------------------------------------------+   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  |  +- v Warnings (93) [expanded] ------------------------------------------------+   |         |
|  |  |                                                                             |   |         |
|  |  |  Type                   | Message (truncated)                     | File           |
|  |  |  -----------------------+-----------------------------------------+----------------|
|  |  |  subscriptions_invalid  | Publisher IED not found: @              | MU1_0P3.cid    |
|  |  |  lgos_orphan            | LGOS configured for 'UPA_3T4_G03' but.. | DUPC_0B.CID    |
|  |  |  lsvs_orphan            | LSVS configured for 'MU_OLD_SV01'...    | PUPC_3L1.CID   |
|  |  |  goose_shared_mac       | 2 GOOSE streams share same dest MAC..   | MU2_3L1.CID    |
|  |  |  sv_shared_mac          | 2 SV streams share same dest MAC..      | MU1_3L1.CID    |
|  |  |  (click row to open detail drawer)                                          |   |         |
|  |  |                                                                             |   |         |
|  |  |  < 1  2  3 ... 19 >    Showing 1-5 of 93                                   |   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  |  +- > Info (3,095) [collapsed] ------------------------------------------------+   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  +--------------------------------------------------------------------------------------+         |
+--------------------------------------------------------------------------------------------------+
```

---

#### Diagnostics accordion -- Info expanded

```
+--------------------------------------------------------------------------------------------------+
|  +- Inline expansion: SubstationXYZ_2026.scd -----------------------------------------+         |
|  |                                                                                      |         |
|  |  [Summary]  [Diagnostics]                                                            |         |
|  |             =============                                                            |         |
|  |                                                                                      |         |
|  |  [! 38 Errors]  [! 93 Warnings]  [o 3,095 Info]                                     |         |
|  |                                                                                      |         |
|  |  +- > Errors (38) [collapsed] -------------------------------------------------+   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  |  +- > Warnings (93) [collapsed] -----------------------------------------------+   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  |  +- v Info (3,095) [expanded] -------------------------------------------------+   |         |
|  |  |                                                                             |   |         |
|  |  |  Type                   | Message (truncated)                     | File           |
|  |  |  -----------------------+-----------------------------------------+----------------|
|  |  |  ieds_proxy             | IED DUCD_3T1 referenced but CID missing | PUPC_3B.CID    |
|  |  |  ieds_proxy             | IED UPA_3T4 referenced but CID missing  | PUPC_3B.CID    |
|  |  |  ieds_duplicates        | DUCD_3T3 found in 2 files, selected..   | DUCD_3T3.CID   |
|  |  |  goose_duplicates       | GoCB01 (DUCD_3T3) found in 3 files..    | MU1_0P3.cid    |
|  |  |  subscriptions_valid    | PUPC_0P3 -> MU1_0P3 CTRL/LLN0 Ind089   | MU1_0P3.CID    |
|  |  |  (click row to open detail drawer)                                          |   |         |
|  |  |                                                                             |   |         |
|  |  |  < 1  2  3 ... 619 >    Showing 1-5 of 3,095                               |   |         |
|  |  +-----------------------------------------------------------------------------+   |         |
|  |                                                                                      |         |
|  +--------------------------------------------------------------------------------------+         |
+--------------------------------------------------------------------------------------------------+
```

---

#### Drawer -- Diagnostic detail (goose_incomplete)

Clicking any row in the diagnostics table opens a right-side drawer with the full diagnostic detail.

```
+--------------------------------------------------+  +---------------------------------+
|                                                  |  | DIAGNOSTIC DETAIL           [X] |
|  (SCD list + expanded row visible behind drawer) |  +---------------------------------+
|                                                  |  |                                 |
|                                                  |  |  Level:   ! Error               |
|                                                  |  |  Type:    goose_incomplete      |
|                                                  |  |                                 |
|                                                  |  |  IED:         UPA_3T4           |
|                                                  |  |  LD:          Master            |
|                                                  |  |  CB Name:     GoCB03            |
|                                                  |  |  Source File: PUPC_3B.CID       |
|                                                  |  |  Subnetwork:  W1                |
|                                                  |  |                                 |
|                                                  |  |  Message:                       |
|                                                  |  |  GOOSE found in Communication   |
|                                                  |  |  but GSEControl not defined in  |
|                                                  |  |  IED -- CID file for UPA_3T4    |
|                                                  |  |  may be missing or incomplete.  |
|                                                  |  |                                 |
|                                                  |  |  Missing Fields:                |
|                                                  |  |  control_block, dataset,        |
|                                                  |  |  goose_id, conf_rev             |
|                                                  |  |                                 |
|                                                  |  |  Known Values:                  |
|                                                  |  |  dest_mac: 01:0c:cd:01:00:fe    |
|                                                  |  |  app_id:   0x00FA               |
|                                                  |  |                                 |
|                                                  |  |  Subscribers (6):               |
|                                                  |  |  UPP_3T3, UPA_3T3, DUPC_0P3,   |
|                                                  |  |  DUPC_0B, DUPC_3B, PUPC_3B     |
|                                                  |  |                                 |
+--------------------------------------------------+  +---------------------------------+
```

---

#### Drawer -- Diagnostic detail (goose_config_mismatch)

```
+--------------------------------------------------+  +---------------------------------+
|                                                  |  | DIAGNOSTIC DETAIL           [X] |
|  (SCD list + expanded row visible behind drawer) |  +---------------------------------+
|                                                  |  |                                 |
|                                                  |  |  Level:   ! Error               |
|                                                  |  |  Type:    goose_config_mismatch |
|                                                  |  |                                 |
|                                                  |  |  IED:     DUCD_3T3              |
|                                                  |  |  LD:      Master                |
|                                                  |  |  CB Name: GoCB04                |
|                                                  |  |                                 |
|                                                  |  |  Issue:                         |
|                                                  |  |  Same control block has 2       |
|                                                  |  |  conflicting configurations     |
|                                                  |  |  across files.                  |
|                                                  |  |                                 |
|                                                  |  |  Suggestion:                    |
|                                                  |  |  Verify which configuration is  |
|                                                  |  |  correct. The most complete was |
|                                                  |  |  selected automatically.        |
|                                                  |  |                                 |
|                                                  |  |  Conflicting Configs:           |
|                                                  |  |  +- Config A (DUPC_0P3.CID) --+ |
|                                                  |  |  |  app_id: 0x0155           | |
|                                                  |  |  |  mac: 01:0c:cd:01:01:55   | |
|                                                  |  |  |  vlan: (none)             | |
|                                                  |  |  +---------------------------+ |
|                                                  |  |  +- Config B [SELECTED] -----+ |
|                                                  |  |  |  Files: DUPC_0B, UPA_3T3, | |
|                                                  |  |  |         DUCD_3T3          | |
|                                                  |  |  |  app_id: 0x015D           | |
|                                                  |  |  |  mac: 01:0c:cd:01:01:5d   | |
|                                                  |  |  |  vlan: 0x0BC9             | |
|                                                  |  |  +---------------------------+ |
+--------------------------------------------------+  +---------------------------------+
```

---

#### Drawer -- Diagnostic detail (lgos_orphan warning)

```
+--------------------------------------------------+  +---------------------------------+
|                                                  |  | DIAGNOSTIC DETAIL           [X] |
|  (SCD list + expanded row visible behind drawer) |  +---------------------------------+
|                                                  |  |                                 |
|                                                  |  |  Level:   ! Warning             |
|                                                  |  |  Type:    lgos_orphan           |
|                                                  |  |                                 |
|                                                  |  |  IED:      DUPC_0B              |
|                                                  |  |  LD:       Master               |
|                                                  |  |  LN:       LGOS28               |
|                                                  |  |  Source:   DUPC_0B.CID          |
|                                                  |  |                                 |
|                                                  |  |  Issue:                         |
|                                                  |  |  LGOS configured for            |
|                                                  |  |  'UPA_3T4_G03' but MAC          |
|                                                  |  |  01:0c:cd:01:00:fe belongs to   |
|                                                  |  |  'UAD_0T3_GO3'.                 |
|                                                  |  |                                 |
|                                                  |  |  Configured stream: UPA_3T4_G03 |
|                                                  |  |  Actual stream:     UAD_0T3_GO3 |
|                                                  |  |  dest_mac: 01:0c:cd:01:00:fe    |
|                                                  |  |  cb_ref: UPA_3T4Master/LLN0...  |
|                                                  |  |                                 |
|                                                  |  |  Suggestion:                    |
|                                                  |  |  Update DUPC_0B.CID to use      |
|                                                  |  |  goose_id 'UAD_0T3_GO3' or      |
|                                                  |  |  verify if the publisher IED    |
|                                                  |  |  was renamed.                   |
|                                                  |  |                                 |
+--------------------------------------------------+  +---------------------------------+
```

---

#### Upload SCD flow

`[Upload SCD]` is visible for ADMIN role only. Clicking it opens an upload dialog.

```
+-----------------------------------------------------+
|  Upload SCD File                              [X]   |
+-----------------------------------------------------+
|                                                     |
|  File type:                                         |
|  (o) SCD -- Substation Configuration Description   |
|  ( ) CID -- Configured IED Description             |
|                                                     |
|  +- Drop zone --------------------------------+     |
|  |                                           |     |
|  |   Drag & drop .scd/.cid file here         |     |
|  |        or  [Browse files]                 |     |
|  |                                           |     |
|  |   Accepted: .scd, .cid  (max 50 MB)      |     |
|  +-------------------------------------------+     |
|                                                     |
|  [Cancel]                        [Upload]           |
+-----------------------------------------------------+
```

**Upload in progress:**

```
+-----------------------------------------------------+
|  Upload SCD File                              [X]   |
+-----------------------------------------------------+
|                                                     |
|  SubstationXYZ_2026b.scd                           |
|  [============================================>    ]|
|  Uploading... 72%                                   |
|                                                     |
|  [Cancel]                                           |
+-----------------------------------------------------+
```

**Upload complete (parse queued):**

```
+-----------------------------------------------------+
|  Upload SCD File                              [X]   |
+-----------------------------------------------------+
|                                                     |
|  * Upload complete.                                 |
|  SubstationXYZ_2026b.scd is being parsed.          |
|  Status will update to PARSED or STALLED            |
|  automatically in a few seconds.                    |
|                                                     |
|  [Close]                                            |
+-----------------------------------------------------+
```

---

#### Inline expansion -- Actions area (varies by SCD status)

The Actions area appears at the bottom of the inline expansion panel. Available actions depend on the SCD status and the user's role.

**Status: PARSED (ADMIN sees Confirm / Reject / Delete)**

```
+- Actions -------------------------------------------------------+
|                                                                 |
|  [Confirm]    [Reject]    [Delete]                              |
|                                                                 |
|  Confirm: activates this SCD as the substation configuration.  |
|  Reject:  requires a reason (see rejection form below).        |
|  Delete:  soft-deletes the SCD record.                         |
|                                                                 |
+-----------------------------------------------------------------+
```

**Rejection form (shown inline after clicking Reject)**

```
+- Reject SCD ---------------------------------------------------+
|                                                                |
|  Reason for rejection (required):                              |
|  +----------------------------------------------------------+  |
|  |  [_____________________________________________]         |  |
|  +----------------------------------------------------------+  |
|                                                                |
|  Additional notes (optional):                                  |
|  +----------------------------------------------------------+  |
|  |  [_____________________________________________]         |  |
|  +----------------------------------------------------------+  |
|                                                                |
|  [Confirm Rejection]    [Cancel]                               |
|                                                                |
+----------------------------------------------------------------+
```

**Delete confirmation (shown inline after clicking Delete)**

```
+- Confirm Delete -----------------------------------------------+
|                                                                |
|  ! Are you sure you want to delete                             |
|  "PUPC_3B_20260201.CID"?                                       |
|  This action cannot be undone.                                 |
|                                                                |
|  [Yes, Delete]    [Cancel]                                     |
|                                                                |
+----------------------------------------------------------------+
```

**Status: STALLED (ADMIN sees Reject / Delete)**

```
+- Actions -------------------------------------------------------+
|                                                                 |
|  [Reject]    [Delete]                                           |
|                                                                 |
|  Parser failed or timed out. Review the error and decide       |
|  whether to reject or delete this entry.                       |
|                                                                 |
+-----------------------------------------------------------------+
```

**Status: UPLOADED (ADMIN sees Delete only)**

```
+- Actions -------------------------------------------------------+
|                                                                 |
|  [Delete]                                                       |
|                                                                 |
|  Awaiting automatic parse -- no other actions available.       |
|                                                                 |
+-----------------------------------------------------------------+
```

**Status: ACTIVE (no actions available)**

```
+- Actions -------------------------------------------------------+
|                                                                 |
|  (No actions available -- active SCD cannot be modified.)      |
|                                                                 |
+-----------------------------------------------------------------+
```

**Status: INACTIVE (ADMIN sees Delete)**

```
+- Actions -------------------------------------------------------+
|                                                                 |
|  [Delete]                                                       |
|                                                                 |
+-----------------------------------------------------------------+
```

**Status: REJECTED or DELETED (no actions available)**

```
+- Actions -------------------------------------------------------+
|                                                                 |
|  (No actions available.)                                        |
|                                                                 |
+-----------------------------------------------------------------+
```

---

#### Empty state -- no SCDs (ADMIN)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header ---------------------------------------------------------------------------------+     |
|  |  [Logo NMS]   --              Mon: o STOPPED                   admin@empresa.com        |     |
|  +----------------------------------------------------------------------------------------  +     |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                 |
| > Sett.  |  =======                                                                               |
|   > SCD  |                                                                                        |
|   > Mon. |                   +-------------------------------------------+                       |
|   > MIB  |                   |                                           |                       |
|          |                   |    (icon: file with gear)                  |                       |
|          |                   |                                           |                       |
|          |                   |   No SCD on record.                       |                       |
|          |                   |                                           |                       |
|          |                   |   Upload an SCD or CID file to            |                       |
|          |                   |   configure the substation.               |                       |
|          |                   |                                           |                       |
|          |                   |   [Upload SCD]                            |                       |
|          |                   |                                           |                       |
|          |                   +-------------------------------------------+                       |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

---

#### Empty state -- no SCDs (VIEWER / OPERATOR)

```
+--------------------------------------------------------------------------------------------------+
|  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                            |
|  =======                                                                                          |
|                                                                                                   |
|                   +-------------------------------------------+                                  |
|                   |                                           |                                  |
|                   |    (icon: file with gear)                  |                                  |
|                   |                                           |                                  |
|                   |   No SCD on record.                       |                                  |
|                   |                                           |                                  |
|                   |   Ask an administrator to upload          |                                  |
|                   |   an SCD file.                            |                                  |
|                   |                                           |                                  |
|                   +-------------------------------------------+                                  |
+--------------------------------------------------------------------------------------------------+
```

---

### /settings/monitoring -- Monitoring Control

#### Monitoring state machine

```
+----------+           +----------+           +----------+
| STOPPED  |---------->| STARTING |---------->| RUNNING  |
| (gray)   | Start     | (amber,  | success   | (green)  |
|          |           | pulsing) |           |          |
+----------+           +----+-----+           +-----+----+
     ^                      |                       |
     |                      | failure               | Stop
     |                      v                       v
     |                 +----------+           +----------+
     +--------+--------| STOPPING |<----------| RUNNING  |
               success | (gray,   |           |          |
                       | pulsing) |           +----------+
                       +----+-----+
                            |
                            | failure
                            v
                       +----------+
                       |  ERROR   |
                       | (red)    |
                       +----------+
```

#### Main state -- RUNNING

```
+--------------------------------------------------------------------------------------------------+
|  +- Header ---------------------------------------------------------------------------------+     |
|  |  [Logo NMS]   SubstationXYZ   Mon: * RUNNING                   admin@empresa.com        |     |
|  +----------------------------------------------------------------------------------------  +     |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                 |
| > Sett.  |             =============                                                              |
|   > SCD  |                                                                                        |
|   > Mon. |  +- Monitoring Control ----------------------------------------------------------+     |
|   > MIB  |  |                                                                               |     |
|          |  |  +- Status card (full-width) -------------------------------------------+    |     |
|          |  |  |                                                                       |    |     |
|          |  |  |  * RUNNING                                                           |    |     |
|          |  |  |  Since: 27/03/2026 10:00                                             |    |     |
|          |  |  |  Started by: admin@empresa.com                                       |    |     |
|          |  |  |                                                                       |    |     |
|          |  |  |  [Stop Monitoring]                                                   |    |     |
|          |  |  |  (visible for ADMIN and OPERATOR)                                    |    |     |
|          |  |  |                                                                       |    |     |
|          |  |  +-----------------------------------------------------------------------+    |     |
|          |  |                                                                               |     |
|          |  |  +- Monitoring items --------------------------------------------------- +    |     |
|          |  |  |                                                                        |    |     |
|          |  |  |  Protocol | Target                   | Status   | Last Update         |    |     |
|          |  |  |  ---------+--------------------------+----------+---------------------|    |     |
|          |  |  |  SV       | LDTM1                    | running  | 27/03/2026 14:32    |    |     |
|          |  |  |  GOOSE    | GoCB01                   | running  | 27/03/2026 14:32    |    |     |
|          |  |  |  PTP      | DUCD_3T3                 | running  | 27/03/2026 14:32    |    |     |
|          |  |  |  MMS      | 192.168.110.104          | running  | 27/03/2026 14:32    |    |     |
|          |  |  |  SNMP     | 192.168.110.105          | running  | 27/03/2026 14:32    |    |     |
|          |  |  |                                                                        |    |     |
|          |  |  +------------------------------------------------------------------------+    |     |
|          |  |                                                                               |     |
|          |  +-------------------------------------------------------------------------------+     |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

---

#### Main state -- STOPPED

```
+--------------------------------------------------------------------------------------------------+
|  +- Header ---------------------------------------------------------------------------------+     |
|  |  [Logo NMS]   SubstationXYZ   Mon: o STOPPED                   admin@empresa.com        |     |
|  +----------------------------------------------------------------------------------------  +     |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                 |
| > Sett.  |             =============                                                              |
|   > SCD  |                                                                                        |
|   > Mon. |  +- Monitoring Control ----------------------------------------------------------+     |
|   > MIB  |  |                                                                               |     |
|          |  |  +- Status card (full-width) -------------------------------------------+    |     |
|          |  |  |                                                                       |    |     |
|          |  |  |  o STOPPED                                                           |    |     |
|          |  |  |  Since: 27/03/2026 15:30                                             |    |     |
|          |  |  |  Stopped by: admin@empresa.com                                       |    |     |
|          |  |  |                                                                       |    |     |
|          |  |  |  [Start Monitoring]                                                  |    |     |
|          |  |  |  (visible for ADMIN and OPERATOR)                                    |    |     |
|          |  |  |                                                                       |    |     |
|          |  |  +-----------------------------------------------------------------------+    |     |
|          |  |                                                                               |     |
|          |  |  +- Monitoring items -------------------------------------------------------+    |     |
|          |  |  |  (empty -- no active monitoring items when stopped)                      |    |     |
|          |  |  +-------------------------------------------------------------------------+    |     |
|          |  |                                                                               |     |
|          |  +-------------------------------------------------------------------------------+     |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

---

#### State -- STARTING (transitional)

```
+- Status card -------------------------------------------------------------------+
|                                                                                 |
|  * STARTING (pulsing indicator)                                                 |
|  Initiating monitoring engine...                                                |
|                                                                                 |
|  [Stop] (disabled during transition)                                            |
|                                                                                 |
+---------------------------------------------------------------------------------+
```

---

#### State -- ERROR

```
+- Status card -------------------------------------------------------------------+
|                                                                                 |
|  ! ERROR                                                                        |
|  Since: 27/03/2026 16:00                                                        |
|  The monitoring engine encountered an error.                                    |
|  Check system logs for details.                                                 |
|                                                                                 |
|  [Retry]   [Stop]                                                               |
|                                                                                 |
+---------------------------------------------------------------------------------+
```

---

#### Stop monitoring -- confirmation dialog

```
+-----------------------------------------------+
|  Stop Monitoring                          [X]  |
+-----------------------------------------------+
|                                                |
|  ! Are you sure you want to stop monitoring?  |
|                                                |
|  All active protocol monitors will be          |
|  stopped. Alarms will no longer be generated   |
|  until monitoring is restarted.                |
|                                                |
|  [Confirm Stop]    [Cancel]                    |
+-----------------------------------------------+
```

---

### /settings/mib -- MIB Management

#### Main state (with MIBs loaded)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header ---------------------------------------------------------------------------------+     |
|  |  [Logo NMS]   SubstationXYZ   Mon: * RUNNING                   admin@empresa.com        |     |
|  +----------------------------------------------------------------------------------------  +     |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                 |
| > Sett.  |                               =======                                                  |
|   > SCD  |                                                                                        |
|   > Mon. |  +- MIB Management ----------------------------------------------------------+         |
|   > MIB  |  |                                                           [Upload MIB]    |         |
|          |  |  [Search: ________________________________________]  [Clear]              |         |
|          |  |                                                                            |         |
|          |  |  Name               | Version | Description                   | Created         |
|          |  |  -------------------+---------+-------------------------------+-----------------|
|          |  |  GE-MIB             | 1.2.3   | GE equipment base OIDs        | 12/01/2026 09:00|
|          |  |  GE-VERNOVA-IED-MIB | 2.0.1   | GE Vernova IED extensions     | 12/01/2026 09:05|
|          |  |  (click row to open detail drawer)                                         |
|          |  |                                                                            |         |
|          |  |  < Prev   Page 1 of 1   Next >    Showing 2 of 2                          |         |
|          |  +------------------------------------------------------------------------------------+  |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

**Table behavior:**

- Click a row to open the MIB detail drawer on the right.
- `[Upload MIB]` button is visible for ADMIN only.
- Search filters by MIB name.

---

#### Drawer -- MIB detail

```
+--------------------------------------------------+  +---------------------------------+
|                                                  |  | MIB DETAIL                  [X] |
|  (MIB list visible behind drawer)                |  +---------------------------------+
|                                                  |  |                                 |
|                                                  |  |  Name:     GE-MIB               |
|                                                  |  |  Version:  1.2.3                |
|                                                  |  |                                 |
|                                                  |  |  Description:                   |
|                                                  |  |  GE equipment base OIDs for     |
|                                                  |  |  all GE substation devices.     |
|                                                  |  |                                 |
|                                                  |  |  Created:  12/01/2026 09:00     |
|                                                  |  |  Updated:  12/01/2026 09:00     |
|                                                  |  |                                 |
|                                                  |  |  Content preview:               |
|                                                  |  |  +---------------------------+  |
|                                                  |  |  | GE-MIB DEFINITIONS       |  |
|                                                  |  |  | IMPLICIT TAGS ::= BEGIN   |  |
|                                                  |  |  | IMPORTS ...               |  |
|                                                  |  |  | (truncated at 10 lines)   |  |
|                                                  |  |  +---------------------------+  |
|                                                  |  |                                 |
|                                                  |  |  [Delete]                       |
|                                                  |  |  (ADMIN only)                   |
|                                                  |  |                                 |
+--------------------------------------------------+  +---------------------------------+
```

---

#### Delete MIB -- confirmation

Clicking `[Delete]` in the MIB drawer replaces the delete button with an inline confirmation:

```
|  +- Confirm Delete -----------------------+  |
|  |                                        |  |
|  |  ! Delete "GE-MIB"?                    |  |
|  |  This action cannot be undone.         |  |
|  |                                        |  |
|  |  [Yes, Delete]    [Cancel]             |  |
|  |                                        |  |
|  +----------------------------------------+  |
```

---

#### Upload MIB flow

Clicking `[Upload MIB]` opens a dialog:

```
+-----------------------------------------------------+
|  Upload MIB File                              [X]   |
+-----------------------------------------------------+
|                                                     |
|  Name (required):                                   |
|  +-----------------------------------------------+  |
|  |  [____________________________________]       |  |
|  +-----------------------------------------------+  |
|                                                     |
|  Version (optional):                                |
|  +-----------------------------------------------+  |
|  |  [____________________________________]       |  |
|  +-----------------------------------------------+  |
|                                                     |
|  Description (optional):                            |
|  +-----------------------------------------------+  |
|  |  [____________________________________]       |  |
|  +-----------------------------------------------+  |
|                                                     |
|  +- Drop zone --------------------------------+     |
|  |                                           |     |
|  |   Drag & drop .mib file here              |     |
|  |        or  [Browse files]                 |     |
|  |                                           |     |
|  +-------------------------------------------+     |
|                                                     |
|  [Cancel]                        [Upload]           |
+-----------------------------------------------------+
```

---

#### Empty state -- no MIBs (ADMIN)

```
+--------------------------------------------------------------------------------------------------+
|  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                            |
|                               =======                                                             |
|                                                                                                   |
|                   +-------------------------------------------+                                  |
|                   |                                           |                                  |
|                   |    (icon: document with OID)              |                                  |
|                   |                                           |                                  |
|                   |   No MIB files registered.               |                                  |
|                   |                                           |                                  |
|                   |   Upload a MIB file to enable SNMP        |                                  |
|                   |   OID translation.                        |                                  |
|                   |                                           |                                  |
|                   |   [Upload MIB]                            |                                  |
|                   |                                           |                                  |
|                   +-------------------------------------------+                                  |
+--------------------------------------------------------------------------------------------------+
```

---

#### Empty state -- no MIBs (VIEWER / OPERATOR)

```
+--------------------------------------------------------------------------------------------------+
|  [ SCD ]    [ Monitoring ]    [ MIBs ]                                                            |
|                               =======                                                             |
|                                                                                                   |
|                   +-------------------------------------------+                                  |
|                   |                                           |                                  |
|                   |    (icon: document with OID)              |                                  |
|                   |                                           |                                  |
|                   |   No MIB files registered.               |                                  |
|                   |                                           |                                  |
|                   |   Ask an administrator to upload          |                                  |
|                   |   MIB files.                              |                                  |
|                   |                                           |                                  |
|                   +-------------------------------------------+                                  |
+--------------------------------------------------------------------------------------------------+
```

---

## Components

| Component | Description | Data Source |
|-----------|-------------|-------------|
| **Settings tab bar** | Shared layout component rendered on all three sub-routes. Each tab is a `<Link>`. Active tab determined by current pathname. | Static (pathname) |
| **SCD table** | Paginated table of SCD records with status badge, dates, and row click to expand inline detail. Default sort: `createdAtUtc` desc. | `GET /api/v1/scds` |
| **SCD inline expansion** | Detail panel rendered below the clicked row. Contains two inner tabs: Summary and Diagnostics. | `GET /api/v1/scds/{scdId}/summary`, `GET /api/v1/scds/{scdId}/VALIDATIONS` |
| **SCD summary metrics grid** | Grid of metric boxes inside the Summary tab: IEDs (real/proxy), Streams (GOOSE/SV), Subscriptions (valid/invalid), MMS Points (total/LGOS/LSVS/LTMS), Vendors, Source files. | `GET /api/v1/scds/{scdId}/summary` |
| **Diagnostics badge row** | Row of three badges summarizing diagnostics counts: `[! N Errors] [! N Warnings] [o N Info]`. Static from summary endpoint. | `GET /api/v1/scds/{scdId}/summary` -> `diagnostics` |
| **Diagnostics accordion** | Three collapsible sections (Errors, Warnings, Info). Errors expanded by default. Each section contains a paginated table of diagnostics. Lazy loaded from VALIDATIONS endpoint. | `GET /api/v1/scds/{scdId}/VALIDATIONS` |
| **Diagnostic detail drawer** | Right-side drawer opened by clicking a row in the diagnostics table. Shows full message, suggestion, affected IED/CB, source file, missing fields. For config_mismatch: shows conflicting configs side by side. | data already loaded from VALIDATIONS |
| **SCD actions area** | Bottom of expansion. Buttons vary by SCD status and role. Includes inline rejection form and delete confirmation. | Role + SCD status |
| **Upload SCD dialog** | Modal with file type selector (SCD/CID), drag-and-drop zone, progress bar, and success/error feedback. | `POST /api/v1/scds` |
| **Monitoring status card** | Full-width card showing current state (`* RUNNING` / `o STOPPED` / `! ERROR`) with start/stop button for ADMIN and OPERATOR. | `GET /api/v1/monitoring` |
| **Monitoring items table** | List of active monitoring items with protocol, target, status and last update timestamp. | `GET /api/v1/monitoring` -> `monitorings[]` |
| **Stop monitoring dialog** | Confirmation dialog before stopping the monitoring engine. | UI only |
| **MIB table** | Paginated table of MIB files with name, version, description and creation date. Row click opens MIB detail drawer. | `GET /api/v1/mibs` |
| **MIB detail drawer** | Right-side drawer with full MIB metadata and content preview. Delete button for ADMIN. | `GET /api/v1/mibs/{mibId}` |
| **Upload MIB dialog** | Modal with name, version, description inputs and file drop zone. | `POST /api/v1/mibs` |
| **Empty state** | Centered card with illustration, message and optional CTA button when a list is empty. Text varies by role. | Derived from empty API response |
| **API error banner** | Inline red banner at top of content area when any endpoint returns HTTP 5xx or network error. | Any failing endpoint |

---

## Data and Endpoints

| # | Method | Endpoint | Usage | Fields | Notes |
|---|--------|----------|-------|--------|-------|
| 1 | `GET` | `/api/v1/scds` | SCD list with pagination and optional status filter | `scdId`, `fileName`, `status`, `createdAtUtc`, `parsedAtUtc`, `activatedAtUtc` | Query params: `status`, `q`, `page`, `pageSize`, `order` |
| 2 | `GET` | `/api/v1/scds/{scdId}/summary` | Enriched SCD summary for inline expansion Summary tab | `substationName`, `status`, `parsedAtUtc`, `activatedAtUtc`, `sourceFiles`, `summary.*` | Cached -- fetched once per scdId |
| 3 | `GET` | `/api/v1/scds/{scdId}/VALIDATIONS` | Full diagnostics for Diagnostics accordion | `data.error.*`, `data.warning.*`, `data.info.*` | Lazy loaded when Diagnostics tab first opened. Cached per scdId. |
| 4 | `POST` | `/api/v1/scds` | Upload new SCD/CID file | `file` (multipart), `type` (SCD/CID) | ADMIN only. Returns `scdId` and initial status `UPLOADED`. |
| 5 | `POST` | `/api/v1/scds/{scdId}/confirmations` | Confirm (activate) or reject a PARSED SCD | `action` (CONFIRM/REJECT), `reason` (required for REJECT) | ADMIN only |
| 6 | `DELETE` | `/api/v1/scds/{scdId}` | Soft-delete an SCD | -- | ADMIN only. Not available for ACTIVE status. |
| 7 | `GET` | `/api/v1/monitoring` | Monitoring engine status and items list | `state`, `sinceUtc`, `stoppedAtUtc`, `stoppedBy`, `monitorings[]` | Polled every 15 seconds on this page |
| 8 | `PATCH` | `/api/v1/monitoring` | Start or stop the monitoring engine | `enabled` (boolean) | ADMIN and OPERATOR. `enabled: true` to start, `false` to stop. |
| 9 | `GET` | `/api/v1/mibs` | Paginated MIB list | `mibId`, `name`, `version`, `description`, `createdAtUtc` | Query params: `q`, `page`, `pageSize` |
| 10 | `GET` | `/api/v1/mibs/{mibId}` | Full MIB detail including content | `mibId`, `name`, `version`, `description`, `content`, `createdAtUtc`, `updatedAtUtc` | Fetched when MIB drawer opens |
| 11 | `POST` | `/api/v1/mibs` | Create a new MIB entry | `name`, `version`, `description`, `content` | ADMIN only. Returns 409 if name already exists. |
| 12 | `DELETE` | `/api/v1/mibs/{mibId}` | Delete a MIB | -- | ADMIN only |

### Loading strategy

```
> Page load /settings/scd
   GET /api/v1/scds (page 1, all statuses)
   +- Render SCD table
   +- User clicks a row
       GET /api/v1/scds/{scdId}/summary     --> populate Summary tab
       (Diagnostics tab not yet loaded)
       +- User clicks Diagnostics tab
           GET /api/v1/scds/{scdId}/VALIDATIONS  --> lazy load accordion
           +- Errors section expanded by default
           +- Cache VALIDATIONS per scdId (stale-while-revalidate)

> Page load /settings/monitoring
   GET /api/v1/monitoring   --> render status card + items table
   Start polling every 15 seconds

> Page load /settings/mib
   GET /api/v1/mibs (page 1)
   +- Render MIB table
   +- User clicks a row
       GET /api/v1/mibs/{mibId}   --> render MIB drawer
```

### Relevant schemas

| Schema | Key Fields | Notes |
|--------|-----------|-------|
| `ScdRecord` | `scdId`, `fileName`, `sha256`, `status` (ScdStatus), `createdAtUtc`, `parsedAtUtc`, `activatedAtUtc` | Used in SCD table |
| `ScdStatus` | `UPLOADED`, `PARSED`, `STALLED`, `ACTIVE`, `INACTIVE`, `REJECTED`, `DELETED` | Enum |
| `ScdSummaryResponse` | `scdId`, `status`, `substationName`, `parsedAtUtc`, `sourceFiles[]`, `summary` (nested) | Summary tab data |
| `ScdViewEnum` | `SV`, `GOOSE`, `IEDS`, `NETWORKS`, `VALIDATIONS` | View selector for `GET /scds/{scdId}/{view}` |
| `ScdProtocolDetailsResponse` | `scdId`, `status`, `substationName`, `data` (varies by view) | Used for VALIDATIONS view |
| `MonitoringStatusResponse` | `enabled`, `state` (MonitoringState), `sinceUtc`, `monitorings[]`, `stoppedAtUtc`, `stoppedBy` | Monitoring page |
| `MonitoringState` | `STOPPED`, `STARTING`, `RUNNING`, `STOPPING`, `ERROR` | Enum |
| `MonitoringItem` | `id`, `protocol`, `target`, `status`, `updatedAtUtc` | Row in monitoring items table |
| `Mib` | `mibId`, `name`, `version`, `description`, `content`, `createdAtUtc`, `updatedAtUtc` | MIB table and drawer |
| `CreateMibRequest` | `name`, `version`, `description`, `content` | Upload MIB payload |
| `PageMeta` | `page`, `pageSize`, `total` | Pagination metadata |

---

## Interaction Flows

### 1. Page load /settings/scd

```
> User navigates to /settings/scd
> Settings parent item in sidebar auto-expands (SCD child is active)
> SCD tab is highlighted with bottom border
> Frontend executes GET /api/v1/scds (page 1)
> If response is empty:
      > Show empty state (Upload CTA for ADMIN, info message for others)
> If response has items:
      > Render SCD table sorted by createdAtUtc desc
      > No row is expanded by default
```

### 2. Expand SCD inline

```
> User clicks a row in the SCD table
> If another row was expanded: collapse it first
> Expanded row shows chevron-down indicator
> Frontend executes GET /api/v1/scds/{scdId}/summary
> While loading: show skeleton in expansion panel
> On success: render Summary tab with metric grid and actions area
> Summary tab is active by default; Diagnostics tab is not yet loaded
```

### 3. Switch to Diagnostics tab

```
> User clicks [Diagnostics] tab in the expansion panel
> If VALIDATIONS not yet loaded for this scdId:
      > Frontend executes GET /api/v1/scds/{scdId}/VALIDATIONS
      > While loading: show spinner in accordion area
      > On success: cache VALIDATIONS, render accordion
> If VALIDATIONS already cached: render immediately
> Errors section is expanded by default
> Warnings and Info sections are collapsed
```

### 4. Open diagnostic detail drawer

```
> User clicks a row in any diagnostics table (Errors, Warnings, or Info)
> Drawer slides in from the right
> Drawer content renders from already-loaded VALIDATIONS data (no new request)
> If diagnostic type is goose_config_mismatch or sv_config_mismatch:
      > Show conflicting config comparison panel
> User closes drawer: click [X] or click outside
```

### 5. Upload SCD

```
> ADMIN clicks [Upload SCD]
> Upload dialog opens
> User selects file type (SCD or CID) and chooses a file
> User clicks [Upload]
> Frontend POST /api/v1/scds (multipart form data)
> While uploading: progress bar in dialog
> On success (201):
      > Show "Upload complete, parsing in progress" message
      > Close dialog (or user clicks Close)
      > Invalidate SCD list cache
      > Refetch GET /api/v1/scds
      > New row appears with status UPLOADED
      > After a few seconds the status may auto-update to PARSED or STALLED
        (via polling or manual refresh -- polling interval: 10 seconds when
        any UPLOADED row exists in the current page)
> On error (4xx/5xx): show inline error message in dialog
```

### 6. Confirm SCD (PARSED -> ACTIVE)

```
> ADMIN expands a PARSED SCD row
> Actions area shows [Confirm] [Reject] [Delete]
> ADMIN clicks [Confirm]
> Frontend POST /api/v1/scds/{scdId}/confirmations { action: "CONFIRM" }
> On success:
      > Status badge updates to ACTIVE (green)
      > Previously ACTIVE SCD (if any) updates to INACTIVE
      > Header substation name updates to reflect new SCD
      > Actions area shows "No actions available"
> On error: show inline error message in actions area
```

### 7. Reject SCD

```
> ADMIN clicks [Reject] in actions area
> Rejection form appears inline (replaces action buttons)
> ADMIN fills in reason (required) and optional notes
> ADMIN clicks [Confirm Rejection]
> Frontend POST /api/v1/scds/{scdId}/confirmations { action: "REJECT", reason: "..." }
> On success: status badge updates to REJECTED
> On error: show inline error message in form
> ADMIN can cancel at any time with [Cancel]
```

### 8. Start monitoring

```
> ADMIN or OPERATOR is on /settings/monitoring with STOPPED state
> Clicks [Start Monitoring]
> Frontend PATCH /api/v1/monitoring { enabled: true }
> Status card immediately shows * STARTING (pulsing)
> Polling resumes (15 seconds interval)
> When GET /api/v1/monitoring returns state: "RUNNING":
      > Status card updates to * RUNNING
      > Since timestamp and items table update
      > Header monitoring badge updates to * RUNNING
```

### 9. Stop monitoring

```
> ADMIN or OPERATOR clicks [Stop Monitoring]
> Confirmation dialog appears
> User clicks [Confirm Stop]
> Frontend PATCH /api/v1/monitoring { enabled: false }
> Status card shows o STOPPING (pulsing)
> When GET /api/v1/monitoring returns state: "STOPPED":
      > Status card updates to o STOPPED
      > Items table clears
      > Header monitoring badge updates to o STOPPED
```

### 10. Upload MIB

```
> ADMIN clicks [Upload MIB]
> Upload dialog opens with name, version, description inputs and file drop zone
> User fills in required name, optionally version and description
> User selects or drops a .mib file
> Clicks [Upload]
> Frontend POST /api/v1/mibs { name, version, description, content }
> On success (201): close dialog, refetch GET /api/v1/mibs
> On 409 (name conflict): show "A MIB with this name already exists" inline error
```

### 11. Delete MIB

```
> ADMIN opens MIB detail drawer
> Clicks [Delete]
> Inline confirmation appears in drawer
> ADMIN clicks [Yes, Delete]
> Frontend DELETE /api/v1/mibs/{mibId}
> On success (204): close drawer, remove row from table, show success toast
> On error: show inline error in drawer
```

---

## States

### /settings/scd states

| State | Condition | Visual |
|-------|-----------|--------|
| **SCD list loaded** | `GET /scds` returns items | SCD table with rows. No row expanded. |
| **Row expanded -- Summary** | User clicked a row, summary loaded | Expansion panel below row with metric grid and actions. |
| **Row expanded -- Diagnostics** | User clicked Diagnostics tab, VALIDATIONS loaded | Accordion with Errors expanded, Warnings/Info collapsed. |
| **Diagnostics loading** | VALIDATIONS request in flight | Spinner in accordion area. |
| **Upload in progress** | `POST /scds` in flight | Progress bar in upload dialog. |
| **Awaiting parse** | SCD has status UPLOADED | Row shows `o UPLOADED`. Polling every 10s while any UPLOADED row visible. |
| **Empty** | `GET /scds` returns empty | Empty state card. Upload CTA for ADMIN. |
| **Loading** | Initial request in flight | Skeleton table. |
| **API error** | Any endpoint returns error | Inline red banner at top of content area. |

### /settings/monitoring states

| State | Condition | Visual |
|-------|-----------|--------|
| **RUNNING** | `state: "RUNNING"` | `* RUNNING` in green, since timestamp, items table, `[Stop Monitoring]` button. |
| **STOPPED** | `state: "STOPPED"` | `o STOPPED` in gray, stopped-since timestamp, empty items table, `[Start Monitoring]` button. |
| **STARTING** | `state: "STARTING"` | `* STARTING` pulsing in green, buttons disabled. |
| **STOPPING** | `state: "STOPPING"` | `o STOPPING` pulsing in gray, buttons disabled. |
| **ERROR** | `state: "ERROR"` | `! ERROR` in red, error message, `[Retry]` and `[Stop]` buttons. |
| **Loading** | Initial request in flight | Skeleton status card. |

### /settings/mib states

| State | Condition | Visual |
|-------|-----------|--------|
| **MIB list loaded** | `GET /mibs` returns items | MIB table with rows. |
| **Drawer open** | User clicked a row | MIB detail drawer on right side. |
| **Delete confirmation** | User clicked Delete in drawer | Inline confirmation in drawer. |
| **Upload dialog open** | User clicked Upload MIB | Upload dialog modal. |
| **Empty** | `GET /mibs` returns empty | Empty state card. Upload CTA for ADMIN. |
| **Loading** | Initial request in flight | Skeleton table. |

---

## Permissions by Role

| Element | ADMIN | OPERATOR | VIEWER |
|---------|-------|----------|--------|
| View SCD list | yes | yes | yes |
| View SCD inline expansion (Summary + Diagnostics) | yes | yes | yes |
| Upload SCD `[Upload SCD]` | yes | no | no |
| Confirm SCD `[Confirm]` | yes | no | no |
| Reject SCD `[Reject]` | yes | no | no |
| Delete SCD `[Delete]` | yes | no | no |
| View Monitoring status and items | yes | yes | yes |
| Start Monitoring `[Start Monitoring]` | yes | yes | no |
| Stop Monitoring `[Stop Monitoring]` | yes | yes | no |
| View MIB list | yes | yes | yes |
| View MIB detail drawer | yes | yes | yes |
| Upload MIB `[Upload MIB]` | yes | no | no |
| Delete MIB `[Delete]` (in drawer) | yes | no | no |

**Notes:**

- OPERATOR can start and stop the monitoring engine but cannot manage SCDs or MIBs.
- VIEWER is read-only on all three sub-routes -- no action buttons are shown.
- For VIEWER and OPERATOR on empty states, the Upload CTA is hidden and replaced with an informational message.

---

## Example Data (for Figma reproduction)

The JSON examples below represent the exact API responses used to populate each screen. Use these values when creating Figma mockups to ensure consistency with the implementation.

---

### Endpoint 1 -- GET /api/v1/scds

Returns the paginated SCD list.

```json
{
  "items": [
    {
      "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "fileName": "SubstationXYZ_2026.scd",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
      "status": "ACTIVE",
      "createdAtUtc": "2026-03-26T12:15:00Z",
      "parsedAtUtc": "2026-03-26T12:15:42Z",
      "activatedAtUtc": "2026-03-26T12:20:00Z"
    },
    {
      "scdId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
      "fileName": "PUPC_3B_20260201.CID",
      "sha256": "abc123def456abc123def456abc123def456abc123def456abc123def456abc1",
      "status": "PARSED",
      "createdAtUtc": "2026-03-28T17:30:00Z",
      "parsedAtUtc": "2026-03-28T17:30:58Z",
      "activatedAtUtc": null
    },
    {
      "scdId": "c3d4e5f6-a7b8-9012-cdef-123456789012",
      "fileName": "DUCD_3T3.CID",
      "sha256": "def456abc123def456abc123def456abc123def456abc123def456abc123def4",
      "status": "STALLED",
      "createdAtUtc": "2026-03-25T12:15:00Z",
      "parsedAtUtc": null,
      "activatedAtUtc": null
    },
    {
      "scdId": "d4e5f6a7-b8c9-0123-defa-234567890123",
      "fileName": "SubstationXYZ_2025.scd",
      "sha256": "f6a7b8c9d0e1f6a7b8c9d0e1f6a7b8c9d0e1f6a7b8c9d0e1f6a7b8c9d0e1f6",
      "status": "INACTIVE",
      "createdAtUtc": "2026-02-15T11:00:00Z",
      "parsedAtUtc": "2026-02-15T11:02:00Z",
      "activatedAtUtc": "2026-02-15T11:10:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 4
  }
}
```

---

### Endpoint 2 -- GET /api/v1/scds/{scdId}/summary

Enriched SCD summary. Populates the Summary tab in the inline expansion.

```json
{
  "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "ACTIVE",
  "substationName": "SubstationXYZ",
  "parsedAtUtc": "2026-03-26T12:15:42Z",
  "activatedAtUtc": "2026-03-26T12:20:00Z",
  "sourceFiles": [
    "SubstationXYZ_2026.scd",
    "DUCD_3T3.CID",
    "MU1_0P3.cid",
    "DUPC_3L1.CID",
    "MU2_3T3.cid",
    "DUPC_0B.CID",
    "PUPC_3P1.CID"
  ],
  "summary": {
    "ieds": 42,
    "ieds_proxy": 161,
    "goose_streams": 237,
    "sv_streams": 80,
    "subnets": 9,
    "type": {
      "mu": 16,
      "ied": 20,
      "other": 6,
      "proxy": 161
    },
    "mms_points": {
      "total": 5370,
      "lgos": 4880,
      "lsvs": 408,
      "ltms": 82
    },
    "subscriptions": {
      "total": 2682,
      "valid": 2682,
      "invalid": 0
    },
    "diagnostics": {
      "info": 3095,
      "warning": 93,
      "error": 38
    },
    "vendors": ["GE", "GE Multilin"],
    "source_files": 42
  }
}
```

**Field-to-card mapping:**

| Card | JSON Fields |
|------|-------------|
| IEDs | `summary.ieds` = 42, `summary.ieds_proxy` = 161, `summary.type` = mu:16, ied:20, other:6, proxy:161 |
| Streams | `summary.goose_streams` = 237, `summary.sv_streams` = 80 |
| Subscriptions | `summary.subscriptions.valid` = 2,682, `summary.subscriptions.invalid` = 0 |
| MMS Points | `summary.mms_points.total` = 5,370, lgos = 4,880, lsvs = 408, ltms = 82 |
| Vendors & Files | `summary.vendors` = GE, GE Multilin; `summary.source_files` = 42 |
| Diagnostics badges | `summary.diagnostics.error` = 38, `.warning` = 93, `.info` = 3,095 |

---

### Endpoint 3 -- GET /api/v1/scds/{scdId}/VALIDATIONS

Full diagnostics detail. Loaded lazily when the Diagnostics tab is first opened.

```json
{
  "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "ACTIVE",
  "substationName": "SubstationXYZ",
  "data": {
    "error": {
      "goose_incomplete": [
        {
          "publisher_ied": "UPA_3T4",
          "ld_inst": "Master",
          "cb_name": "GoCB03",
          "dest_mac": "01:0c:cd:01:00:fe",
          "app_id": "0x00FA",
          "vlan_id": "",
          "subnetwork": "W1",
          "subscribers": ["UPP_3T3", "UPA_3T3", "DUPC_0P3", "DUPC_0B", "DUPC_3B", "PUPC_3B"],
          "source_file": "PUPC_3B.CID",
          "missing_fields": ["control_block", "dataset", "goose_id", "conf_rev"],
          "message": "GOOSE found in Communication but GSEControl not defined in IED - CID file for UPA_3T4 may be missing or incomplete"
        },
        {
          "publisher_ied": "MU2_0IB1MU0",
          "ld_inst": "2",
          "cb_name": "MU2_0IB1",
          "dest_mac": "01:0c:cd:04:01:72",
          "app_id": "0x4000",
          "vlan_id": "",
          "subnetwork": "W4",
          "subscribers": [],
          "source_file": "DUPC_0P3.CID",
          "missing_fields": ["sv_id", "conf_rev"],
          "message": "SV found in Communication but SampledValueControl not defined in IED - CID file for MU2_0IB1MU0 may be missing or incomplete"
        }
      ],
      "goose_config_mismatch": [
        {
          "publisher_ied": "DUCD_3T3",
          "ld_inst": "Master",
          "cb_name": "GoCB04",
          "configs": [
            {
              "goose_id": "DUCD_3T3_GO4",
              "app_id": "0x0155",
              "dest_mac": "01:0c:cd:01:01:55",
              "vlan_id": "",
              "vlan_priority": "",
              "conf_rev": 1,
              "source_files": ["DUPC_0P3.CID"]
            },
            {
              "goose_id": "DUCD_3T3_GO4",
              "app_id": "0x015D",
              "dest_mac": "01:0c:cd:01:01:5d",
              "vlan_id": "0x0BC9",
              "vlan_priority": 5,
              "conf_rev": 1,
              "source_files": ["DUPC_0B.CID", "UPA_3T3.CID", "DUCD_3T3.CID"]
            }
          ],
          "selected": {
            "goose_id": "DUCD_3T3_GO4",
            "app_id": "0x015D",
            "dest_mac": "01:0c:cd:01:01:5d",
            "vlan_id": "0x0BC9",
            "vlan_priority": 5,
            "source_file": "DUCD_3T3.CID"
          },
          "issue": "Same control block has 2 conflicting configurations across files",
          "suggestion": "Verify which configuration is correct. The most complete was selected."
        }
      ],
      "sv_config_mismatch": [
        {
          "publisher_ied": "RxSV2MU02",
          "ld_inst": "1",
          "cb_name": "MSVCB03",
          "configs": [
            {
              "sv_id": "MU2_3P10201",
              "app_id": "0x4000",
              "dest_mac": "01:0c:cd:04:01:4c",
              "vlan_id": "",
              "vlan_priority": "",
              "conf_rev": 1,
              "source_files": ["DUCD_3T3.CID", "DUPC_3P1.CID"]
            },
            {
              "sv_id": "MU2_3P10201",
              "app_id": "0x4000",
              "dest_mac": "01:0c:cd:04:01:4b",
              "vlan_id": "",
              "vlan_priority": "",
              "conf_rev": 1,
              "source_files": ["PUCD_3T3.CID", "PUPC_3P1.CID"]
            }
          ],
          "selected": {
            "sv_id": "MU2_3P10201",
            "app_id": "0x4000",
            "dest_mac": "01:0c:cd:04:01:4c",
            "vlan_id": "",
            "vlan_priority": "",
            "source_file": "DUCD_3T3.CID"
          },
          "issue": "Same control block has 2 conflicting configurations across files",
          "suggestion": "Verify which configuration is correct. The most complete was selected."
        }
      ],
      "sv_incomplete": [
        {
          "publisher_ied": "MU2_0IB1MU0",
          "ld_inst": "2",
          "cb_name": "MU2_0IB1",
          "dest_mac": "01:0c:cd:04:01:72",
          "app_id": "0x4000",
          "vlan_id": "",
          "subnetwork": "W4",
          "subscribers": [],
          "source_file": "DUPC_0P3.CID",
          "missing_fields": ["sv_id", "conf_rev"],
          "message": "SV found in Communication but SampledValueControl not defined in IED - CID file for MU2_0IB1MU0 may be missing or incomplete"
        }
      ]
    },
    "warning": {
      "subscriptions_invalid": [
        {
          "extref": {
            "subscriber_ied": "MU1_0P3",
            "subscriber_ld": "LDSUIED",
            "subscriber_ln_class": "LPDO",
            "subscriber_ln_inst": "1",
            "subscriber_ln_prefix": "",
            "publisher_ied": "@",
            "publisher_ld": "LDDJ",
            "publisher_ln_class": "XCMD",
            "publisher_ln_inst": "0",
            "publisher_ln_prefix": "",
            "do_name": "OpOpn",
            "da_name": "general",
            "src_cb_name": "",
            "desc": ""
          },
          "error": "Publisher IED not found: @",
          "reason": "publisher_ied_not_found"
        }
      ],
      "lgos_orphan": [
        {
          "ied": "DUPC_0B",
          "ld": "Master",
          "ln": "LGOS28",
          "configured_stream_id": "UPA_3T4_G03",
          "dest_mac": "01:0c:cd:01:00:fe",
          "cb_ref": "UPA_3T4Master/LLN0.GoCB03",
          "source_file": "DUPC_0B.CID",
          "actual_goose_id": "UAD_0T3_GO3",
          "issue": "LGOS configured for 'UPA_3T4_G03' but MAC 01:0c:cd:01:00:fe belongs to 'UAD_0T3_GO3'",
          "suggestion": "Update DUPC_0B.CID to use goose_id 'UAD_0T3_GO3' or verify if the publisher IED was renamed"
        }
      ],
      "lsvs_orphan": [
        {
          "ied": "PUPC_3L1",
          "ld": "PBus",
          "ln": "LSVS3",
          "configured_stream_id": "MU_OLD_SV01",
          "dest_mac": "01:0c:cd:04:00:10",
          "cb_ref": "MU_OLDLDTM1/LLN0.MSVCB01",
          "source_file": "PUPC_3L1.CID",
          "issue": "LSVS configured for 'MU_OLD_SV01' which does not exist in any loaded CID",
          "suggestion": "Check if the publisher IED CID file is missing or if 'MU_OLD_SV01' was removed"
        }
      ],
      "goose_shared_mac": [
        {
          "dest_mac": "01:0c:cd:01:00:23",
          "stream_count": 2,
          "streams": [
            {
              "publisher_ied": "MU2_3L1",
              "control_block": "MU2_3L1CTRL/LLN0$GO$FastGOOSE1",
              "goose_id": "MU2_3L1_G01"
            },
            {
              "publisher_ied": "PUPC_3L1",
              "control_block": "PUPC_3L1Master/LLN0$GO$GoCB01",
              "goose_id": "PUPC_3L1_G01"
            }
          ],
          "issue": "2 GOOSE streams share same destination MAC",
          "suggestion": "Consider assigning unique MAC per stream for better multicast filtering"
        }
      ],
      "sv_shared_mac": [
        {
          "dest_mac": "01:0c:cd:04:00:01",
          "stream_count": 2,
          "streams": [
            {
              "publisher_ied": "MU1_3L1",
              "sv_id": "MU1_3L1_SV01"
            },
            {
              "publisher_ied": "MU2_3L1",
              "sv_id": "MU2_3L1_SV01"
            }
          ],
          "issue": "2 SV streams share same destination MAC",
          "suggestion": "Consider assigning unique MAC per stream for better multicast filtering"
        }
      ]
    },
    "info": {
      "ieds_duplicates": [
        {
          "ied_name": "DUCD_3T3",
          "selected": {
            "source_file": "DUCD_3T3.CID",
            "logical_devices": 7
          },
          "duplicates": [
            {
              "source_file": "MU1_0P3.cid",
              "logical_devices": 3
            }
          ]
        }
      ],
      "ieds_proxy": [
        {
          "ied_name": "DUCD_3T1",
          "desc": "GE_Digital_Energy_UR URPC_GOOSE_Proxy",
          "type": "GE_Digital_Energy_UR URPC",
          "source_file": "PUPC_3B.CID"
        },
        {
          "ied_name": "UPA_3T4",
          "desc": "GE_Digital_Energy_UR",
          "type": "GE_Digital_Energy_UR",
          "source_file": "PUPC_3B.CID"
        }
      ],
      "goose_duplicates": [
        {
          "publisher_ied": "DUCD_3T3",
          "ld_inst": "Master",
          "cb_name": "GoCB01",
          "dest_mac": "01:0c:cd:01:00:00",
          "goose_id": "TxGOOSE1",
          "source_file": "DUCD_3T3.CID",
          "also_in": [
            {
              "source_file": "MU1_0P3.cid"
            },
            {
              "source_file": "PUPC_0P3.CID"
            }
          ]
        }
      ],
      "goose_duplicates": [
        {
          "publisher_ied": "MU1_3L2",
          "ld_inst": "MU01",
          "cb_name": "MU1_3L2",
          "dest_mac": "01:0c:cd:04:00:89",
          "sv_id": "MU1_3L20101",
          "source_file": "MU1_3L2.cid",
          "also_in": [
            {
              "source_file": "PUPC_3L2.CID"
            }
          ]
        }
      ],
      "subscriptions_valid": [
        {
          "subscriber_ied": "MU1_0P3",
          "subscriber_ld": "CTRL",
          "subscriber_ln_class": "LLN0",
          "subscriber_ln_inst": "",
          "subscriber_ln_prefix": "",
          "publisher_ied": "PUPC_0P3",
          "publisher_ld": "Gen",
          "publisher_ln_class": "GAPC",
          "publisher_ln_inst": "1",
          "publisher_ln_prefix": "FlxLgc",
          "do_name": "Ind089",
          "da_name": "stVal",
          "src_cb_name": "GoCB02",
          "desc": ""
        }
      ]
    }
  }
}
```

**Diagnostics counts (from summary):**

| Level | Count | Shown in |
|-------|-------|---------|
| Errors | 38 | `[! 38 Errors]` badge, Errors accordion header |
| Warnings | 93 | `[! 93 Warnings]` badge, Warnings accordion header |
| Info | 3,095 | `[o 3,095 Info]` badge, Info accordion header |

---

### Endpoint 4 -- GET /api/v1/monitoring

Monitoring engine status. State RUNNING:

```json
{
  "enabled": true,
  "state": "RUNNING",
  "sinceUtc": "2026-03-27T13:00:00Z",
  "monitorings": [
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-111111111111",
      "protocol": "SV",
      "target": "LDTM1",
      "status": "running",
      "updatedAtUtc": "2026-03-27T17:32:00Z"
    },
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-222222222222",
      "protocol": "GOOSE",
      "target": "GoCB01",
      "status": "running",
      "updatedAtUtc": "2026-03-27T17:32:00Z"
    },
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-333333333333",
      "protocol": "PTP",
      "target": "DUCD_3T3",
      "status": "running",
      "updatedAtUtc": "2026-03-27T17:32:00Z"
    },
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-444444444444",
      "protocol": "MMS",
      "target": "192.168.110.104",
      "status": "running",
      "updatedAtUtc": "2026-03-27T17:32:00Z"
    },
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-555555555555",
      "protocol": "SNMP",
      "target": "192.168.110.105",
      "status": "running",
      "updatedAtUtc": "2026-03-27T17:32:00Z"
    }
  ],
  "stoppedAtUtc": null,
  "stoppedBy": null
}
```

State STOPPED:

```json
{
  "enabled": false,
  "state": "STOPPED",
  "sinceUtc": "2026-03-27T18:30:00Z",
  "monitorings": [],
  "stoppedAtUtc": "2026-03-27T18:30:00Z",
  "stoppedBy": "admin@empresa.com"
}
```

**Figma values for RUNNING state:**

- Status indicator: `* RUNNING` (green)
- Since: `27/03/2026 10:00` (sinceUtc `2026-03-27T13:00:00Z` converted to BRT UTC-3)
- Items table: 5 rows as above

**Figma values for STOPPED state:**

- Status indicator: `o STOPPED` (gray)
- Since: `27/03/2026 15:30` (stoppedAtUtc `2026-03-27T18:30:00Z` converted to BRT UTC-3)
- Stopped by: `admin@empresa.com`

---

### Endpoint 5 -- GET /api/v1/mibs

Paginated MIB list:

```json
{
  "items": [
    {
      "mibId": "e5f6a7b8-c9d0-1234-efab-567890123456",
      "name": "GE-MIB",
      "version": "1.2.3",
      "description": "GE equipment base OIDs for all GE substation devices.",
      "content": "GE-MIB DEFINITIONS IMPLICIT TAGS ::= BEGIN\nIMPORTS ...",
      "createdAtUtc": "2026-01-12T12:00:00Z",
      "updatedAtUtc": "2026-01-12T12:00:00Z"
    },
    {
      "mibId": "f6a7b8c9-d0e1-2345-fabc-678901234567",
      "name": "GE-VERNOVA-IED-MIB",
      "version": "2.0.1",
      "description": "GE Vernova IED-specific OID extensions.",
      "content": "GE-VERNOVA-IED-MIB DEFINITIONS IMPLICIT TAGS ::= BEGIN\nIMPORTS ...",
      "createdAtUtc": "2026-01-12T12:05:00Z",
      "updatedAtUtc": "2026-01-12T12:05:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 2
  }
}
```

---

### Summary table -- all values for Figma

Use this table as a single reference when laying out all three sub-routes in Figma.

**Header (all sub-routes):**

| Field | Value |
|-------|-------|
| Substation name | SubstationXYZ |
| Monitoring status | `* RUNNING` (green) |
| User | admin@empresa.com |

**/settings/scd -- SCD table rows:**

| File Name | Status | Created | Parsed | Activated |
|-----------|--------|---------|--------|-----------|
| SubstationXYZ_2026.scd | `* ACTIVE` (green) | 26/03/2026 09:15 | 26/03/2026 09:16 | 26/03/2026 09:20 |
| PUPC_3B_20260201.CID | `o PARSED` (amber) | 28/03/2026 14:30 | 28/03/2026 14:31 | - |
| DUCD_3T3.CID | `! STALLED` (orange) | 25/03/2026 09:15 | - | - |
| SubstationXYZ_2025.scd | `o INACTIVE` (gray) | 15/02/2026 08:00 | 15/02/2026 08:02 | 15/02/2026 08:10 |

**/settings/scd -- inline expansion (ACTIVE SCD):**

| Metric | Value |
|--------|-------|
| Substation | SubstationXYZ |
| Status | `* ACTIVE` |
| Parsed | 26/03/2026 09:16 |
| Activated | 26/03/2026 09:20 |
| Source files | SubstationXYZ_2026.scd (+41 CID) |
| IEDs Real | 42 |
| IEDs Proxy | 161 |
| Type mu | 16 |
| Type ied | 20 |
| Type other | 6 |
| Type proxy | 161 |
| GOOSE streams | 237 |
| SV streams | 80 |
| Subscriptions valid | 2,682 |
| Subscriptions invalid | 0 |
| MMS Total | 5,370 |
| MMS LGOS | 4,880 |
| MMS LSVS | 408 |
| MMS LTMS | 82 |
| Vendors | GE, GE Multilin |
| Source files (count) | 42 |

**/settings/scd -- diagnostics badges:**

| Level | Count |
|-------|-------|
| Errors | 38 |
| Warnings | 93 |
| Info | 3,095 |

**/settings/scd -- errors table (first page, 5 rows):**

| Type | Message (truncated) | File |
|------|---------------------|------|
| goose_incomplete | GOOSE found in Communication but GSEControl not defined in IED... | PUPC_3B.CID |
| sv_incomplete | SV found in Communication but SampledValueControl not defined... | DUPC_0P3.CID |
| goose_config_mismatch | Same control block has 2 conflicting configurations across files | DUCD_3T3.CID |
| sv_config_mismatch | Same control block has 2 conflicting configurations across files | DUCD_3T3.CID |

**/settings/monitoring -- RUNNING state:**

| Field | Value |
|-------|-------|
| State | `* RUNNING` (green) |
| Since | 27/03/2026 10:00 |
| Items | SV/LDTM1, GOOSE/GoCB01, PTP/DUCD_3T3, MMS/192.168.110.104, SNMP/192.168.110.105 (all running) |
| Button | `[Stop Monitoring]` |

**/settings/mib -- MIB table rows:**

| Name | Version | Description | Created |
|------|---------|-------------|---------|
| GE-MIB | 1.2.3 | GE equipment base OIDs | 12/01/2026 09:00 |
| GE-VERNOVA-IED-MIB | 2.0.1 | GE Vernova IED extensions | 12/01/2026 09:05 |

---

## References

- `00-navegacao-global.md` -- Master layout, header, sidebar, RBAC conventions, ASCII symbol legend
- `09-dashboard.md` -- Dashboard wireframe (references same SCD summary data for dashboard cards)
- `docs/parsers/scd-parser.md` -- Parser SCD documentation: Summary schema, Diagnostics types and JSON examples
- `docs/api/swagger-nms-v1.0.1.yaml` -- REST API spec: `ScdViewEnum` (VALIDATIONS), `MonitoringStatusResponse`, `Mib`, `ScdStatus`, `MonitoringState`
