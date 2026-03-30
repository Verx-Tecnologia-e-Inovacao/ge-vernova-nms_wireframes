# Screen 4 — Network Redundancy (HSR/PRP)

## Objective

Monitor the redundancy status of GOOSE and Sampled Values (SV) streams across network interfaces, using PRP (Parallel Redundancy Protocol) and HSR (High-availability Seamless Redundancy) data reported by the GOOSE and SV parsers. This is a **derived view** — redundancy information is extracted from the `stats.<interface>.prp_lans` and `stats.<interface>.hsr_paths` fields of existing GOOSE/SV metrics, not from dedicated HSR/PRP endpoints.

**Route:** `/redundancy`

The page displays a single table combining GOOSE and SV streams with their redundancy diagnostic per interface. The diagnostic is computed client-side by cross-referencing PRP LANs between eth0 and eth1 (IEC 62439-3 PRP) and checking HSR path visibility on each interface (IEC 62439-3 HSR ring topology).

**Data sources:** `GET /api/v1/protocols/GOOSE/metrics` and `GET /api/v1/protocols/SV/metrics`.

The sample data uses 6 streams covering all diagnostic scenarios: Normal (PRP), Warning (LANs inverted), Error (same LAN on both interfaces), Normal (HSR both paths), and No redundancy (no PRP/HSR detected).

---

## Wireframe

### Main state — active SCD + monitoring RUNNING

```
+---------------------------------------------------------------------------------------------------+
|  +-Header Global (see 00-navegacao-global.md) -----------------------------------------------+    |
|  | [Logo NMS]   Substation: AA1J1     Mon: * RUNNING        {user} [Logout]                  |    |
|  +-------------------------------------------------------------------------------------------+    |
+---------+-----------------------------------------------------------------------------------------+
| SIDEBAR |                                                                                         |
|         | +-Filter Bar -------------------------------------------------------------------+       |
| 1 Dashb | |                                                                               |       |
| > Proto | |  {Protocol v}    {Diagnostic v}                                     [Clear]   |       |
| 3 Alerts| |                                                                               |       |
| 4 Device| |  Protocol options: All, GOOSE, SV                                             |       |
| 5 Topol | |  Diagnostic options: All, Normal, Warning, Error, No redundancy               |       |
| 6 Redund| |                                                                               |       |
| 7 Analyt| +-------------------------------------------------------------------------------+       |
| 8 Log   |                                                                                         |
| > Settin| +-Redundancy Streams (6 records) -----------------------------------------------+       |
|         | |                                                                               |       |
|         | |  Proto ^  Stream ID                 Publisher   PRP LANs  HSR   Diagnostic    |       |
|         | |  -------  -------------------------  ----------  --------  ----  -----------   |      |
|         | |  GOOSE    ..Q01../LLN0$GO$gcbTrip   AA1J1Q01    A | B     -     * Normal      |       |
|         | |  GOOSE    ..Q02../LLN0$GO$gcbStat   AA1J1Q02    B | A     -     ! Warning     |       |
|         | |  GOOSE    ..Q03../LLN0$GO$gcbAlrm   AA1J1Q03    A | A     -     X Error       |       |
|         | |  SV       PROT_IED01/LLN0$MU01      PROT_IED01  A | B     -     * Normal      |       |
|         | |  SV       MU_IED02/LLN0$MU01        MU_IED02    - | -     A,B   * Normal      |       |
|         | |  SV       MU_IED03/LLN0$MU01        MU_IED03    - | -     -     o None        |       |
|         | |                                                                               |       |
|         | +-------------------------------------------------------------------------------+       |
|         |                                                                                         |
|         | < Previous  Page 1 of 1  Next >    Showing 6 of 6                                       |
|         |                                                                                         |
|         | +-Status Legend -----------------------------------------------------------------+      |
|         | |  * Normal: PRP or HSR redundancy operating correctly                          |       |
|         | |  ! Warning: LANs inverted, asymmetric PRP, or possible HSR ring break         |       |
|         | |  X Error: same LAN on both interfaces or topology problem                     |       |
|         | |  o None: no PRP trailer or HSR tag detected on this stream                    |       |
|         | +-------------------------------------------------------------------------------+       |
|         |                                                                                         |
| [User]  |                                                                                         |
+---------+-----------------------------------------------------------------------------------------+
```

**Table details:**

- **Columns:** Protocol, Stream ID, Publisher, PRP LANs (compact eth0|eth1 format), HSR Paths, Diagnostic
- **Default sort:** Protocol (ascending), then Publisher (ascending)
- **Sortable columns:** Protocol, Publisher, Diagnostic
- **PRP LANs display format:** `A | B` means eth0=["A"], eth1=["B"]. The pipe `|` separates eth0 from eth1. `-` means empty array
- **HSR Paths display format:** `A,B` means ["A","B"] detected. `-` means empty array. HSR is typically observed on a single interface (ring topology)
- **Stream ID truncation:** Long IDs are truncated with ellipsis (`..`) and full value shown in tooltip
- **Row click:** Opens the redundancy detail drawer (described below)
- **Muted rows:** Streams with "None" diagnostic are displayed with reduced opacity

---

### Drawer — Normal PRP stream (AA1J1Q01 gcbTrip)

Opens from the right when the user clicks a row. Example with a healthy PRP stream:

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  * gcbTrip - AA1J1Q01                    |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Stream Identification --             |
                              |                                          |
                              |  Protocol:   GOOSE                       |
                              |  Stream ID:  AA1J1Q01A1LD0/              |
                              |              LLN0$GO$gcbTrip              |
                              |  Publisher:  AA1J1Q01                     |
                              |  Dest MAC:   01:0c:cd:01:00:00           |
                              |  Subnet:     W01                         |
                              |                                          |
                              |  -- Redundancy by Interface --           |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Field      | eth0    | eth1     |    |
                              |  +----------------------------------+    |
                              |  | PRP LANs   | A       | B        |    |
                              |  | HSR Paths  | -       | -        |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Diagnostic --                        |
                              |                                          |
                              |  +----------------------------------+    |
                              |  |                                  |    |
                              |  |  * Normal                        |    |
                              |  |                                  |    |
                              |  |  PRP: LAN-A on eth0, LAN-B on   |    |
                              |  |  eth1. Standard dual-LAN         |    |
                              |  |  operation per IEC 62439-3.      |    |
                              |  |                                  |    |
                              |  +----------------------------------+    |
                              |                                          |
                              +------------------------------------------+
```

---

### Drawer — Warning stream (AA1J1Q02 gcbStatus, LANs inverted)

Example with a PRP warning — LANs are swapped between interfaces:

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  ! gcbStatus - AA1J1Q02                  |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Stream Identification --             |
                              |                                          |
                              |  Protocol:   GOOSE                       |
                              |  Stream ID:  AA1J1Q02A1LD0/              |
                              |              LLN0$GO$gcbStatus            |
                              |  Publisher:  AA1J1Q02                     |
                              |  Dest MAC:   01:0c:cd:01:00:01           |
                              |  Subnet:     W01                         |
                              |                                          |
                              |  -- Redundancy by Interface --           |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Field      | eth0    | eth1     |    |
                              |  +----------------------------------+    |
                              |  | PRP LANs   | B       | A        |    |
                              |  | HSR Paths  | -       | -        |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Diagnostic --                        |
                              |                                          |
                              |  +----------------------------------+    |
                              |  |                                  |    |
                              |  |  ! Warning - LANs inverted       |    |
                              |  |                                  |    |
                              |  |  PRP: LAN-B on eth0, LAN-A on   |    |
                              |  |  eth1. Expected LAN-A on eth0    |    |
                              |  |  and LAN-B on eth1. Check cable  |    |
                              |  |  connections.                    |    |
                              |  |                                  |    |
                              |  +----------------------------------+    |
                              |                                          |
                              +------------------------------------------+
```

---

### Drawer — Error stream (AA1J1Q03 gcbAlarm, same LAN on both)

Example with a PRP error — same LAN detected on both interfaces:

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  X gcbAlarm - AA1J1Q03                   |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Stream Identification --             |
                              |                                          |
                              |  Protocol:   GOOSE                       |
                              |  Stream ID:  AA1J1Q03A1LD0/              |
                              |              LLN0$GO$gcbAlarm             |
                              |  Publisher:  AA1J1Q03                     |
                              |  Dest MAC:   01:0c:cd:01:00:02           |
                              |  Subnet:     W02                         |
                              |                                          |
                              |  -- Redundancy by Interface --           |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Field      | eth0    | eth1     |    |
                              |  +----------------------------------+    |
                              |  | PRP LANs   | A       | A        |    |
                              |  | HSR Paths  | -       | -        |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Diagnostic --                        |
                              |                                          |
                              |  +----------------------------------+    |
                              |  |                                  |    |
                              |  |  X Error - Same LAN on both      |    |
                              |  |                                  |    |
                              |  |  PRP: LAN-A detected on both     |    |
                              |  |  eth0 and eth1. Both interfaces  |    |
                              |  |  are receiving from the same     |    |
                              |  |  PRP LAN. This indicates a       |    |
                              |  |  configuration or wiring error.  |    |
                              |  |                                  |    |
                              |  +----------------------------------+    |
                              |                                          |
                              +------------------------------------------+
```

---

### Drawer — No redundancy stream (MU_IED03)

Example with a stream that has no PRP or HSR data:

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  o MU01 - MU_IED03                       |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Stream Identification --             |
                              |                                          |
                              |  Protocol:   SV                          |
                              |  Stream ID:  MU_IED03/LLN0$MU01          |
                              |  Publisher:  MU_IED03                     |
                              |  Dest MAC:   01:0c:cd:04:00:06           |
                              |  Subnet:     W01                         |
                              |                                          |
                              |  -- Redundancy by Interface --           |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Field      | eth0    | eth1     |    |
                              |  +----------------------------------+    |
                              |  | PRP LANs   | -       | -        |    |
                              |  | HSR Paths  | -       | -        |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Diagnostic --                        |
                              |                                          |
                              |  +----------------------------------+    |
                              |  |                                  |    |
                              |  |  o No redundancy                 |    |
                              |  |                                  |    |
                              |  |  No PRP trailer or HSR tag       |    |
                              |  |  detected on this stream. The    |    |
                              |  |  stream may not be configured    |    |
                              |  |  for redundant operation.        |    |
                              |  |                                  |    |
                              |  +----------------------------------+    |
                              |                                          |
                              +------------------------------------------+
```

---

### Empty state — No active SCD

Displayed when `GET /scds?latest=true` returns no SCD with status `ACTIVE`. Replaces the entire content area.

```
+---------------------------------------------------------------------------------------------------+
|  +-Header Global (see 00-navegacao-global.md) -----------------------------------------------+    |
|  | [Logo NMS]   Substation: -             Mon: o STOPPED     {user} [Logout]                 |    |
|  +-------------------------------------------------------------------------------------------+    |
+---------+-----------------------------------------------------------------------------------------+
| SIDEBAR |                                                                                         |
|         |                                                                                         |
| 1 Dashb |                                                                                         |
| > Proto |                                                                                         |
| 3 Alerts|                                                                                         |
| 4 Device|      +-----------------------------------------------------+                            |
| 5 Topol |      |                                                     |                            |
| 6 Redund|      |       (icon: shield with question mark /            |                            |
| 7 Analyt|      |        network redundancy)                          |                            |
| 8 Log   |      |                                                     |                            |
| > Settin|      |    No active SCD.                                   |                            |
|         |      |                                                     |                            |
|         |      |    Redundancy diagnostics require an active          |                           |
|         |      |    SCD file with parsed GOOSE and SV streams.        |                           |
|         |      |    Upload and activate an SCD in Settings.           |                           |
|         |      |                                                     |                            |
|         |      |    < Go to Settings >                                |                           |
|         |      |                                                     |                            |
|         |      +-----------------------------------------------------+                            |
|         |                                                                                         |
| [User]  |                                                                                         |
+---------+-----------------------------------------------------------------------------------------+
```

**Details:**

- The icon should represent network redundancy or protection (shield)
- The message explains that SCD with GOOSE/SV data is required for redundancy diagnostics
- The link `< Go to Settings >` navigates to `/settings/scd` (Screen 7)
- The link visibility depends on role (see Permissions section)

---

### Empty state — Monitoring stopped

Displayed when SCD is active but monitoring status is `STOPPED`. Replaces the table area.

```
+---------------------------------------------------------------------------------------------------+
|  +-Header Global (see 00-navegacao-global.md) -----------------------------------------------+    |
|  | [Logo NMS]   Substation: AA1J1     Mon: o STOPPED       {user} [Logout]                  |     |
|  +-------------------------------------------------------------------------------------------+    |
+---------+-----------------------------------------------------------------------------------------+
| SIDEBAR |                                                                                         |
|         |                                                                                         |
| 1 Dashb |                                                                                         |
| > Proto |                                                                                         |
| 3 Alerts|                                                                                         |
| 4 Device|      +-----------------------------------------------------+                            |
| 5 Topol |      |                                                     |                            |
| 6 Redund|      |       (icon: shield with pause symbol /             |                            |
| 7 Analyt|      |        monitoring inactive)                         |                            |
| 8 Log   |      |                                                     |                            |
| > Settin|      |    Monitoring stopped.                               |                           |
|         |      |                                                     |                            |
|         |      |    Redundancy diagnostics are derived from           |                           |
|         |      |    real-time GOOSE and SV metrics. Start             |                           |
|         |      |    monitoring to view redundancy status.             |                           |
|         |      |                                                     |                            |
|         |      +-----------------------------------------------------+                            |
|         |                                                                                         |
| [User]  |                                                                                         |
+---------+-----------------------------------------------------------------------------------------+
```

**Details:**

- The header shows the substation name but monitoring as `o STOPPED`
- No action link — monitoring start/stop is controlled from the Settings screen
- The message explains that redundancy diagnostics require active monitoring (real-time metrics)

---

## Diagnostic Reference

The redundancy diagnostic is computed **client-side** by the frontend after fetching GOOSE and SV metrics. The diagnostic logic follows the tables below.

### PRP Diagnostic (cross-interface comparison)

PRP uses dual LANs (LAN-A and LAN-B). Each interface should receive from a different LAN. The diagnostic compares `prp_lans` arrays between eth0 and eth1.

| eth0.prp_lans | eth1.prp_lans | Diagnostic | Severity |
|---|---|---|---|
| `["A"]` | `["B"]` | Normal | Normal |
| `["B"]` | `["A"]` | LANs inverted — possible cable swap | Warning |
| `["A","B"]` | `["A","B"]` | Topology problem — both LANs on both interfaces | Error |
| `["A"]` | `["A"]` | Same LAN on both interfaces — configuration error | Error |
| `["B"]` | `["B"]` | Same LAN on both interfaces — configuration error | Error |
| `["A"]` | `[]` | Asymmetric PRP — one interface missing | Warning |
| `[]` | `["B"]` | Asymmetric PRP — one interface missing | Warning |
| `[]` | `[]` | No PRP detected | No redundancy |

### HSR Diagnostic (single-interface check)

HSR uses a ring topology where frames travel in both directions around the ring. Both paths (A and B) are visible on any single interface, so checking one interface is sufficient — unlike PRP which requires cross-interface comparison. The diagnostic checks the `hsr_paths` array.

| eth0.hsr_paths | Diagnostic | Severity |
|---|---|---|
| `["A","B"]` | Normal — both ring paths visible | Normal |
| `["A"]` or `["B"]` | Possible ring break — only one path visible | Warning |
| `[]` | No HSR detected | No redundancy |

### Combined Diagnostic Priority

When a stream has both PRP and HSR data, the highest severity is used:

**Error > Warning > Normal > No redundancy**

If a stream has no PRP data and no HSR data (`prp_lans=[]` and `hsr_paths=[]` on all interfaces), the diagnostic is **No redundancy** and the row is displayed with reduced opacity (muted).

---

## Components

| Component | Description | Data Source |
|---|---|---|
| **Filter bar** | Two dropdowns: Protocol (All/GOOSE/SV) and Diagnostic (All/Normal/Warning/Error/No redundancy). Clear button resets both. | Client-side state |
| **Redundancy streams table** | Combined table of GOOSE + SV streams. Columns: Protocol, Stream ID, Publisher, PRP LANs (compact eth0\|eth1), HSR Paths, Diagnostic. Sortable by Protocol, Publisher, Diagnostic. | `GET /protocols/GOOSE/metrics` + `GET /protocols/SV/metrics` — fields: `goose_id`/`sv_id`, `publisher`, `stats.<iface>.prp_lans`, `stats.<iface>.hsr_paths` |
| **Redundancy detail drawer** | Right panel (~40% width). Three sections: Stream Identification (protocol, stream_id, publisher, dest_mac, subnet), Redundancy by Interface (eth0\|eth1 table with prp_lans and hsr_paths), Diagnostic (status + explanatory message). | Same metrics endpoints — stream-level detail |
| **Pagination** | Standard pagination per `00-navegacao-global.md` section 6.1. Shows "Page X of Y" and "Showing N of TOTAL". | Client-side over fetched data |
| **Status legend** | Box below pagination explaining the 4 diagnostic indicators: `*` Normal, `!` Warning, `X` Error, `o` None. | Static |
| **Empty state (no SCD)** | Centered box with shield icon, message, and `< Go to Settings >` link to `/settings/scd`. Link hidden for VIEWER role. | `GET /scds?latest=true` returns no ACTIVE SCD |
| **Empty state (monitoring stopped)** | Centered box with shield+pause icon and message. No action link. | Monitoring status = STOPPED |
| **Loading skeleton** | Shimmer placeholders in table area while data loads. | Request in progress |
| **Error banner** | Inline error banner at top of content area. Red background, error icon, message, close button. | HTTP 4xx/5xx or network error |

---

## Data and Endpoints

### Endpoints used

| # | Method | Endpoint | Usage | Fields extracted |
|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds?latest=true` | Check for active SCD (blocking) | `scdId`, `status` |
| 2 | `GET` | `/api/v1/protocols/GOOSE/metrics` | GOOSE streams with redundancy data | `items[].goose_id`, `items[].publisher`, `items[].dest_mac`, `items[].subnet`, `items[].stats.eth0.prp_lans`, `items[].stats.eth0.hsr_paths`, `items[].stats.eth1.prp_lans`, `items[].stats.eth1.hsr_paths` |
| 3 | `GET` | `/api/v1/protocols/SV/metrics` | SV streams with redundancy data | `items[].sv_id`, `items[].publisher`, `items[].dest_mac`, `items[].subnet`, `items[].stats.eth0.prp_lans`, `items[].stats.eth0.hsr_paths`, `items[].stats.eth1.prp_lans`, `items[].stats.eth1.hsr_paths` |

**Note:** The `prp_lans` and `hsr_paths` fields are documented in the parser specs (`docs/extras-2026-03-16/`) but are not yet present in `docs/api/swagger-nms-v1.0.1.yaml`. The Swagger spec will need to be updated to include these fields in the `stats` object when the redundancy feature is implemented.

### Data loading strategy

```
> Step 1 — SCD check (blocking)
   GET /scds?latest=true
   +- If no active SCD > show empty state (end of flow)
   +- If active SCD > check monitoring status

> Step 2 — Monitoring status check
   +- If STOPPED > show monitoring stopped state (end of flow)
   +- If RUNNING > proceed to fetch metrics

> Step 3 — Fetch metrics (parallel, non-blocking)
   GET /protocols/GOOSE/metrics
   GET /protocols/SV/metrics
   +- For each stream, extract stats.eth0 and stats.eth1 redundancy fields
   +- Apply PRP diagnostic table (cross-interface comparison)
   +- Apply HSR diagnostic table (single-interface check)
   +- Derive severity: Error > Warning > Normal > No redundancy
   +- Render combined table sorted by Protocol, then Publisher

> Step 4 — Polling (every 15-30 seconds)
   GET /protocols/GOOSE/metrics
   GET /protocols/SV/metrics
   +- Recompute diagnostics
   +- Update table and drawer (if open) without full re-render
```

### Client-side cache

- SCD status (step 1) can be cached with long TTL — does not change frequently
- Metrics (steps 3-4) are fetched fresh on each polling cycle
- Diagnostic computation is stateless — pure function of current metrics

### Exemplos de JSON (para designs Figma)

The examples below represent realistic data to populate the visual components. The redundancy page is a **derived view** — it extracts `prp_lans` and `hsr_paths` from GOOSE/SV metrics endpoints. The JSON structures below reflect the data as it comes from those endpoints, focused on the redundancy-relevant fields.

#### Exemplo: Linha da tabela — stream GOOSE com PRP (redundancia Normal)

A single GOOSE stream with PRP operating normally: LAN-A on eth0, LAN-B on eth1.

```json
{
  "control_block": "AA1J1Q01A1LD0/LLN0$GO$gcbTrip",
  "app_id": "0x0001",
  "dataset": "AA1J1Q01A1LD0/LLN0$TripDataset",
  "dataset_entries": 8,
  "goose_id": "AA1J1Q01A1_Trip",
  "dest_mac": "01:0c:cd:01:00:01",
  "vlan_id": "0x0064",
  "vlan_priority": 4,
  "conf_rev": 1,
  "ttl": 2000,
  "interfaces": ["eth0", "eth1"],
  "stats": {
    "eth0": {
      "prp_lans": ["A"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1024,
        "lost_packets": 0,
        "duplicates": 0
      }
    },
    "eth1": {
      "prp_lans": ["B"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1024,
        "lost_packets": 0,
        "duplicates": 0
      }
    }
  }
}
```

**Derived fields for the table row:**

| Field | Value |
|---|---|
| Protocol | GOOSE |
| Stream ID | AA1J1Q01A1LD0/LLN0$GO$gcbTrip |
| Publisher | AA1J1Q01 |
| PRP LANs | A \| B |
| HSR Paths | - |
| Diagnostic | * Normal |

#### Exemplo: Linha da tabela — stream SV sem redundancia

An SV stream with no PRP trailer or HSR tag detected on any interface.

```json
{
  "sv_id": "MU_IED03/LLN0$MU01",
  "app_id": "0x4003",
  "dest_mac": "01:0c:cd:04:00:06",
  "vlan_id": ["0x0001"],
  "vlan_priority": [4],
  "dataset": "MU_IED03/LLN0$MSVCB01",
  "dataset_entries": 8,
  "conf_rev": [1],
  "smp_synch": ["local_clock"],
  "sampling_rate": 4800,
  "nominal_freq": 60,
  "interfaces": ["eth0"],
  "stats": {
    "eth0": {
      "prp_lans": [],
      "hsr_paths": [],
      "counters": {
        "total_samples": 9600,
        "lost_packets": 0,
        "duplicates": 0
      }
    }
  }
}
```

**Derived fields for the table row:**

| Field | Value |
|---|---|
| Protocol | SV |
| Stream ID | MU_IED03/LLN0$MU01 |
| Publisher | MU_IED03 |
| PRP LANs | - \| - |
| HSR Paths | - |
| Diagnostic | o None |

#### Exemplo: Drawer completo — PRP Normal (AA1J1Q01 gcbTrip)

Full drawer data for a healthy PRP stream. Includes stream identification, redundancy-by-interface table, and diagnostic.

```json
{
  "control_block": "AA1J1Q01A1LD0/LLN0$GO$gcbTrip",
  "app_id": "0x0001",
  "dataset": "AA1J1Q01A1LD0/LLN0$TripDataset",
  "dataset_entries": 8,
  "goose_id": "AA1J1Q01A1_Trip",
  "dest_mac": "01:0c:cd:01:00:01",
  "vlan_id": "0x0064",
  "vlan_priority": 4,
  "conf_rev": 1,
  "ttl": 2000,
  "quality_bits": ["good"],
  "simulated": [false],
  "clock_failure": [false],
  "clock_not_synchronized": [false],
  "interfaces": ["eth0", "eth1"],
  "stats": {
    "eth0": {
      "prp_lans": ["A"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1024,
        "number_of_events": 3,
        "lost_packets": 0,
        "lost_packets_ts": null,
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null,
        "ied_restarts": 0,
        "ied_restarts_ts": null,
        "missed_events": 0,
        "missed_events_ts": null
      },
      "timing": {
        "min_time": 0.001,
        "max_time": 1.002,
        "transfer_time": 0.001245
      }
    },
    "eth1": {
      "prp_lans": ["B"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1024,
        "number_of_events": 3,
        "lost_packets": 0,
        "lost_packets_ts": null,
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null,
        "ied_restarts": 0,
        "ied_restarts_ts": null,
        "missed_events": 0,
        "missed_events_ts": null
      },
      "timing": {
        "min_time": 0.001,
        "max_time": 1.003,
        "transfer_time": 0.001312
      }
    }
  },
  "_redundancy_diagnostic": {
    "protocol": "PRP",
    "severity": "Normal",
    "label": "Normal",
    "message": "PRP: LAN-A on eth0, LAN-B on eth1. Standard dual-LAN operation per IEC 62439-3."
  }
}
```

**Drawer sections derived from this data:**

- **Stream Identification:** Protocol=GOOSE, Stream ID=AA1J1Q01A1LD0/LLN0$GO$gcbTrip, Publisher=AA1J1Q01, Dest MAC=01:0c:cd:01:00:01, Subnet=W01
- **Redundancy by Interface:** eth0.prp_lans=["A"], eth1.prp_lans=["B"], both hsr_paths=[]
- **Diagnostic:** * Normal -- PRP: LAN-A on eth0, LAN-B on eth1

#### Exemplo: Drawer — PRP Warning (LANs invertidas)

PRP stream with LANs swapped between interfaces. AA1J1Q02 gcbStatus.

```json
{
  "control_block": "AA1J1Q02A1LD0/LLN0$GO$gcbStatus",
  "app_id": "0x0002",
  "dataset": "AA1J1Q02A1LD0/LLN0$StatusDataset",
  "dataset_entries": 16,
  "goose_id": "AA1J1Q02A1_Status",
  "dest_mac": "01:0c:cd:01:00:02",
  "vlan_id": "0x0064",
  "vlan_priority": 4,
  "conf_rev": 2,
  "ttl": 2000,
  "quality_bits": ["good"],
  "simulated": [false],
  "clock_failure": [false],
  "clock_not_synchronized": [false],
  "interfaces": ["eth0", "eth1"],
  "stats": {
    "eth0": {
      "prp_lans": ["B"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1020,
        "number_of_events": 5,
        "lost_packets": 2,
        "lost_packets_ts": "2026-03-14T10:00:03.456789Z",
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null,
        "ied_restarts": 0,
        "ied_restarts_ts": null,
        "missed_events": 0,
        "missed_events_ts": null
      },
      "timing": {
        "min_time": 0.001,
        "max_time": 1.005,
        "transfer_time": 0.001567
      }
    },
    "eth1": {
      "prp_lans": ["A"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1022,
        "number_of_events": 5,
        "lost_packets": 0,
        "lost_packets_ts": null,
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null,
        "ied_restarts": 0,
        "ied_restarts_ts": null,
        "missed_events": 0,
        "missed_events_ts": null
      },
      "timing": {
        "min_time": 0.001,
        "max_time": 1.004,
        "transfer_time": 0.001489
      }
    }
  },
  "_redundancy_diagnostic": {
    "protocol": "PRP",
    "severity": "Warning",
    "label": "LANs inverted",
    "message": "PRP: LAN-B on eth0, LAN-A on eth1. Expected LAN-A on eth0 and LAN-B on eth1. Check cable connections."
  }
}
```

#### Exemplo: Drawer — PRP Error (mesma LAN em ambas interfaces)

PRP stream with the same LAN detected on both interfaces. AA1J1Q03 gcbAlarm.

```json
{
  "control_block": "AA1J1Q03A1LD0/LLN0$GO$gcbAlarm",
  "app_id": "0x0003",
  "dataset": "AA1J1Q03A1LD0/LLN0$AlarmDataset",
  "dataset_entries": 12,
  "goose_id": "AA1J1Q03A1_Alarm",
  "dest_mac": "01:0c:cd:01:00:03",
  "vlan_id": "0x0064",
  "vlan_priority": 4,
  "conf_rev": 1,
  "ttl": 2000,
  "quality_bits": ["good"],
  "simulated": [false],
  "clock_failure": [false],
  "clock_not_synchronized": [false],
  "interfaces": ["eth0", "eth1"],
  "stats": {
    "eth0": {
      "prp_lans": ["A"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1018,
        "number_of_events": 2,
        "lost_packets": 0,
        "lost_packets_ts": null,
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null,
        "ied_restarts": 0,
        "ied_restarts_ts": null,
        "missed_events": 0,
        "missed_events_ts": null
      },
      "timing": {
        "min_time": 0.001,
        "max_time": 1.001,
        "transfer_time": 0.001123
      }
    },
    "eth1": {
      "prp_lans": ["A"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 1016,
        "number_of_events": 2,
        "lost_packets": 0,
        "lost_packets_ts": null,
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null,
        "ied_restarts": 0,
        "ied_restarts_ts": null,
        "missed_events": 0,
        "missed_events_ts": null
      },
      "timing": {
        "min_time": 0.001,
        "max_time": 1.002,
        "transfer_time": 0.001198
      }
    }
  },
  "_redundancy_diagnostic": {
    "protocol": "PRP",
    "severity": "Error",
    "label": "Same LAN on both",
    "message": "PRP: LAN-A detected on both eth0 and eth1. Both interfaces are receiving from the same PRP LAN. This indicates a configuration or wiring error."
  }
}
```

#### Exemplo: Drawer — HSR Normal (ambos paths)

SV stream on an HSR ring topology with both paths visible. MU_IED02.

```json
{
  "sv_id": "MU_IED02/LLN0$MU01",
  "app_id": "0x4002",
  "dest_mac": "01:0c:cd:04:00:05",
  "vlan_id": ["0x0001"],
  "vlan_priority": [4],
  "dataset": "MU_IED02/LLN0$MSVCB01",
  "dataset_entries": 8,
  "conf_rev": [1],
  "smp_synch": ["local_clock"],
  "sampling_rate": 4800,
  "nominal_freq": 60,
  "quality_bits": ["good"],
  "simulated": [false],
  "interfaces": ["eth0"],
  "stats": {
    "eth0": {
      "prp_lans": [],
      "hsr_paths": ["A", "B"],
      "counters": {
        "total_samples": 19200,
        "lost_packets": 0,
        "lost_packets_ts": null,
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null
      },
      "packet_interval": {
        "nominal_us": 208.33,
        "min_us": 205.12,
        "min_ts": "2026-03-14T10:00:01.234567Z",
        "max_us": 212.45,
        "max_ts": "2026-03-14T10:00:03.456789Z",
        "avg_us": 208.31,
        "std_us": 1.24
      }
    }
  },
  "_redundancy_diagnostic": {
    "protocol": "HSR",
    "severity": "Normal",
    "label": "Normal",
    "message": "HSR: Both ring paths (A and B) visible on eth0. Normal ring operation per IEC 62439-3."
  }
}
```

#### Exemplo: Drawer — Sem redundancia

SV stream with no PRP or HSR data detected. MU_IED03.

```json
{
  "sv_id": "MU_IED03/LLN0$MU01",
  "app_id": "0x4003",
  "dest_mac": "01:0c:cd:04:00:06",
  "vlan_id": ["0x0001"],
  "vlan_priority": [4],
  "dataset": "MU_IED03/LLN0$MSVCB01",
  "dataset_entries": 8,
  "conf_rev": [1],
  "smp_synch": ["local_clock"],
  "sampling_rate": 4800,
  "nominal_freq": 60,
  "quality_bits": ["good"],
  "simulated": [false],
  "interfaces": ["eth0"],
  "stats": {
    "eth0": {
      "prp_lans": [],
      "hsr_paths": [],
      "counters": {
        "total_samples": 9600,
        "lost_packets": 0,
        "lost_packets_ts": null,
        "out_of_order": 0,
        "out_of_order_ts": null,
        "duplicates": 0,
        "duplicates_ts": null
      },
      "packet_interval": {
        "nominal_us": 208.33,
        "min_us": 206.00,
        "min_ts": "2026-03-14T10:00:00.500000Z",
        "max_us": 210.50,
        "max_ts": "2026-03-14T10:00:01.000000Z",
        "avg_us": 208.30,
        "std_us": 0.98
      }
    }
  },
  "_redundancy_diagnostic": {
    "protocol": null,
    "severity": "None",
    "label": "No redundancy",
    "message": "No PRP trailer or HSR tag detected on this stream. The stream may not be configured for redundant operation."
  }
}
```

#### Exemplo: Cenarios de diagnostico — resumo compacto

Compact snippets showing the `prp_lans`/`hsr_paths` combinations and the resulting diagnostic label.

**PRP Normal:**
```json
{ "eth0": { "prp_lans": ["A"], "hsr_paths": [] }, "eth1": { "prp_lans": ["B"], "hsr_paths": [] } }
```
Diagnostic: Normal -- LAN-A on eth0, LAN-B on eth1

**PRP LANs inverted:**
```json
{ "eth0": { "prp_lans": ["B"], "hsr_paths": [] }, "eth1": { "prp_lans": ["A"], "hsr_paths": [] } }
```
Diagnostic: Warning -- LANs inverted, possible cable swap

**PRP Same LAN on both:**
```json
{ "eth0": { "prp_lans": ["A"], "hsr_paths": [] }, "eth1": { "prp_lans": ["A"], "hsr_paths": [] } }
```
Diagnostic: Error -- Same LAN on both interfaces, configuration error

**PRP Both LANs on both interfaces:**
```json
{ "eth0": { "prp_lans": ["A", "B"], "hsr_paths": [] }, "eth1": { "prp_lans": ["A", "B"], "hsr_paths": [] } }
```
Diagnostic: Error -- Topology problem, both LANs on both interfaces

**PRP Asymmetric (one interface missing):**
```json
{ "eth0": { "prp_lans": ["A"], "hsr_paths": [] }, "eth1": { "prp_lans": [], "hsr_paths": [] } }
```
Diagnostic: Warning -- Asymmetric PRP, one interface missing

**HSR Normal (both paths):**
```json
{ "eth0": { "prp_lans": [], "hsr_paths": ["A", "B"] } }
```
Diagnostic: Normal -- Both ring paths visible

**HSR Partial ring break:**
```json
{ "eth0": { "prp_lans": [], "hsr_paths": ["A"] } }
```
Diagnostic: Warning -- Only one path visible, possible ring break

**No redundancy:**
```json
{ "eth0": { "prp_lans": [], "hsr_paths": [] }, "eth1": { "prp_lans": [], "hsr_paths": [] } }
```
Diagnostic: No redundancy -- No PRP/HSR detected on this stream

---

## Interaction Flows

### 1. Page load

```
> User navigates to /redundancy
> Frontend executes GET /api/v1/scds?latest=true
> If no active SCD:
     > Show empty state with link < Go to Settings >
     > End of flow
> If active SCD but monitoring STOPPED:
     > Show monitoring stopped state
     > End of flow
> If active SCD and monitoring RUNNING:
     > Show loading skeleton in table area
     > Parallel: GET /protocols/GOOSE/metrics + GET /protocols/SV/metrics
     > For each stream: extract prp_lans and hsr_paths from stats
     > Apply PRP/HSR diagnostic tables
     > Render combined table sorted by Protocol, then Publisher
     > Start polling cycle (every 15-30 seconds)
```

### 2. Apply filter

```
> User selects filter (e.g., Protocol = "GOOSE")
> Table filters client-side (data already loaded)
> Rows that don't match the filter are hidden
> Pagination recalculates total results
> Combine multiple filters (Protocol + Diagnostic)
> [Clear] button resets all filters to "All"
```

### 3. Click row (open drawer)

```
> User clicks a row in the table (e.g., AA1J1Q01 gcbTrip)
> Drawer opens from right with slide-in animation (~40% width)
> Drawer shows:
     Section 1: Stream Identification (protocol, stream_id, publisher, dest_mac, subnet)
     Section 2: Redundancy by Interface (eth0 | eth1 table with prp_lans, hsr_paths)
     Section 3: Diagnostic (severity icon + status + explanatory message)
> Drawer updates on each polling cycle if open
> User clicks [X] or outside drawer to close
> Clicking another row while drawer is open switches to new stream
```

### 4. Periodic metrics refresh (polling)

```
> Every 15-30 seconds:
     > GET /protocols/GOOSE/metrics + GET /protocols/SV/metrics
> Frontend recomputes diagnostics for all streams
> If diagnostic changed for any stream:
     > Update table row visual (status icon, muted/highlighted)
     > If drawer is open for that stream: update drawer content
> No full table re-render — only changed rows update
```

---

## States

### General page states

| State | Condition | Visual |
|---|---|---|
| **With data** | SCD active + monitoring RUNNING + metrics available | Filter bar + table with streams and diagnostics. Rows color-coded by severity |
| **No active SCD** | `GET /scds?latest=true` returns no ACTIVE SCD | Empty state centered: shield icon + message + `< Go to Settings >` link |
| **Monitoring stopped** | SCD active + monitoring STOPPED | Empty state centered: shield+pause icon + message explaining need for active monitoring |
| **No redundancy data** | All streams have `prp_lans=[]` and `hsr_paths=[]` | Table visible with all rows showing "None" diagnostic (muted). Informational message at top: "No redundancy data detected. All streams are operating without PRP or HSR." |
| **Loading** | Requests in progress | Skeleton/shimmer placeholders in table area. Filter bar visible but disabled |
| **API error** | HTTP 5xx or network error | Inline error banner at top: "Error loading redundancy data. Try again." + [Retry] button |

### Per-row states

| State | Condition | Visual |
|---|---|---|
| **Normal** | PRP or HSR diagnostic = Normal | `*` green indicator. Standard row appearance |
| **Warning** | PRP diagnostic = LANs inverted, Asymmetric, or HSR = possible ring break | `!` yellow indicator. Row with subtle yellow background |
| **Error** | PRP diagnostic = same LAN on both, or topology problem | `X` red indicator. Row with subtle red background |
| **No redundancy** | `prp_lans=[]` and `hsr_paths=[]` on all interfaces | `o` gray indicator. Row with reduced opacity (muted) |

### Drawer states

| State | Condition | Visual |
|---|---|---|
| **Open with data** | Stream selected + metrics available | All three sections populated: identification, redundancy table, diagnostic message |
| **Open without metrics** | Stream selected + monitoring stopped or no metrics | Identification section populated. Redundancy and Diagnostic sections show: "Metrics unavailable — monitoring inactive" |
| **Loading** | Drawer opened, data being fetched | Skeleton/shimmer inside drawer |

---

## Permissions by Role

| Element | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| View redundancy table | Yes | Yes | Yes |
| Apply filters | Yes | Yes | Yes |
| Open detail drawer | Yes | Yes | Yes |
| View diagnostic details | Yes | Yes | Yes |
| `< Go to Settings >` link (empty state) | Visible — navigates to `/settings/scd` | Visible — navigates to `/settings/scd` (action buttons disabled there) | **Hidden** — VIEWER cannot configure SCD |

**Notes:**

- This screen is exclusively **read-only** — no write actions (no start/stop, no ACK, no upload)
- The only difference by role is the visibility of the Settings link in the empty state
- All roles have identical access to all viewing features, filters, and drawers

---

## Future Structure

> **This section documents features that require dedicated HSR/PRP API endpoints not yet available.** The content above ("Available Today") uses data already provided by the GOOSE and SV parsers. The features below represent the target vision for when the backend exposes dedicated redundancy endpoints.

### Per-switch/port view

When dedicated endpoints exist, the redundancy page should include a view organized by network switch and port, showing:

- Supervision frames transmitted/received per port
- Ring breaks detected
- Duplicate frames discarded
- Port forwarding status (Forwarding, Blocked, Disabled)

### HSR / PRP tabs

The page should evolve to have two tabs:

- **HSR** — Ring topology view with path A/B status per node
- **PRP** — Dual-LAN view with LAN-A/LAN-B status per node

### Temporal metrics chart

A time-series chart (ECharts) showing redundancy health over time:

- X-axis: time window (last 1h, 6h, 24h)
- Y-axis: count of supervision frames, ring breaks, duplicates
- Useful for detecting intermittent issues

### Hypothetical endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/v1/protocols/HSR/metrics` | HSR-specific metrics (ring status, supervision frames, path health) |
| `GET` | `/api/v1/protocols/HSR/data` | HSR data records (persisted) |
| `GET` | `/api/v1/protocols/PRP/metrics` | PRP-specific metrics (LAN status, supervision frames, duplicate handling) |
| `GET` | `/api/v1/protocols/PRP/data` | PRP data records (persisted) |

**Alternative:** HSR/PRP data via SNMP using vendor-specific MIBs:

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/v1/protocols/SNMP/data?type=HSR` | HSR data from SNMP polling |
| `GET` | `/api/v1/protocols/SNMP/metrics` | SNMP metrics including HSR/PRP OIDs |

### Open questions (preserved from original wireframe)

| # | Question | Impact |
|---|---|---|
| 1 | Will HSR/PRP data come via SNMP from switches? | Defines whether we use existing SNMP endpoints or need new ones |
| 2 | Will `HSR` and `PRP` be added to `ProtocolEnum`? | Defines whether dedicated endpoints will exist |
| 3 | Will the `SYSTEM` protocol be used as generic container for HSR/PRP? | Alternative without API changes |
| 4 | Will the SCD include redundancy configuration in the future? | Defines whether static data overlay will be available |
| 5 | Which switch metrics are relevant? (supervision frames, ring breaks, duplicates discarded) | Defines table/chart fields for the future view |

---

## References

- `00-navegacao-global.md` — Layout master, header, sidebar (item 6: Redundancy at `/redundancy`), drawer, pagination, status indicators, RBAC
- `05-comunicacao-dados.md` — GOOSE and SV communication monitoring. Uses the same metrics endpoints (`/protocols/GOOSE/metrics`, `/protocols/SV/metrics`) — the redundancy page extracts `prp_lans` and `hsr_paths` from the same data
- `docs/extras-2026-03-16/3.2. Documentação GOOSE.md` — GOOSE parser documentation, PRP/HSR diagnostic tables, `stats.<interface>` structure
- `docs/extras-2026-03-16/4.2. Documentação Parser SV.md` — SV parser documentation, PRP/HSR diagnostic tables, `stats.<interface>` structure
- `docs/extras-2026-03-16/3. Exemplo GOOSE.json` — Real GOOSE data with `prp_lans` and `hsr_paths` values
- `docs/extras-2026-03-16/4. Exemplo SV.json` — Real SV data with `prp_lans` and `hsr_paths` values
- `docs/api/swagger-nms-v1.0.1.yaml` — API spec (MetricPointListResponse, ScdStatus)
