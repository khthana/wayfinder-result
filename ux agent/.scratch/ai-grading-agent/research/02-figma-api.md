# 02 — Figma REST API: ดึงอะไรมาตรวจได้บ้าง และเข้าถึงไฟล์ของนักศึกษาได้อย่างไร

Status: DRAFT (กำลังวิจัย — จะอัปเดตเรื่อย ๆ)
วันที่ค้น: 2026-08-05
แหล่งอ้างอิงหลัก: https://developers.figma.com/docs/rest-api/

> หมายเหตุ: `www.figma.com/developers/api` (URL เดิม) ตอนนี้ 301 redirect ไป `developers.figma.com/docs/rest-api/` แล้ว

---

## 1. Base URL และภาพรวม

- Base URL: `https://api.figma.com/v1/`
- (Figma for Government ใช้ `https://api.figma-gov.com`)
- หมวด endpoint ที่มี: Files, Images, File versions, Users, Comments, Projects, Components and styles, Variables, Dev resources, Webhooks, Analytics (activity logs, library analytics)
- อ้างอิง: https://developers.figma.com/docs/rest-api/

---

## 2. `GET /v1/files/:key` — โครงสร้างข้อมูลหลัก

อ้างอิง: https://developers.figma.com/docs/rest-api/file-endpoints/

### Query parameters

| Param | ความหมาย |
|---|---|
| `version` | ระบุ version ID ที่ต้องการ (ย้อนดูเวอร์ชันเก่า) |
| `ids` | comma-separated node IDs — ดึงเฉพาะบาง node |
| `depth` | ความลึกของ tree ที่ traverse (เช่น `depth=2` ได้แค่ pages + top-level objects) |
| `geometry` | ใส่ `paths` เพื่อเอา vector data ด้วย |
| `plugin_data` | plugin IDs หรือ `"shared"` |
| `branch_data` | default `false`; ถ้า true จะได้ metadata ของ branch |

Scope ที่ต้องใช้: `file_content:read` (Tier 1)

### รูปร่าง response (ระดับบนสุด)

```json
{
  "name": "Student Project — Food Delivery App",
  "role": "viewer",
  "lastModified": "2026-07-30T08:12:44Z",
  "editorType": "figma",
  "thumbnailUrl": "https://s3-alpha.figma.com/thumbnails/...",
  "version": "1234567890",
  "document": { "id": "0:0", "name": "Document", "type": "DOCUMENT", "children": [] },
  "components": {},
  "componentSets": {},
  "schemaVersion": 0,
  "styles": {},
  "mainFileKey": "…",
  "branches": []
}
```

โครงสร้าง tree: root เป็น `DOCUMENT` → ลูกเป็น `CANVAS` (แต่ละ canvas = 1 page ใน Figma) → ลูกต่อไปเป็น layers
อ้างอิง: https://developers.figma.com/docs/rest-api/files/

### `GET /v1/files/:key/nodes`

- `ids` **required**; คืน node ที่ระบุพร้อม subtree
- response:

```json
{
  "name": "…", "role": "…", "lastModified": "…", "editorType": "figma",
  "thumbnailUrl": "…", "err": null,
  "nodes": {
    "1:23": {
      "document": { "id": "1:23", "type": "FRAME", "children": [] },
      "components": {}, "componentSets": {},
      "schemaVersion": 0, "styles": {}
    }
  }
}
```

- ถ้า node id ไม่มีจริง ค่าใน `nodes` จะเป็น `null`

---

## 3. Node properties ที่ใช้วัดเชิงปริมาณได้

อ้างอิง: https://developers.figma.com/docs/rest-api/file-node-types/ และ https://developers.figma.com/docs/rest-api/files/

### Global properties (มีทุก node)

`id`, `name`, `visible` (default true), `type`, `rotation`, `pluginData`, `sharedPluginData`, `componentPropertyReferences`, `boundVariables`, `explicitVariableModes`

### FRAME

- `layoutMode`: `NONE | HORIZONTAL | VERTICAL | GRID` ← ตรวจว่าใช้ auto-layout ไหม
- `itemSpacing` (default 0)
- `paddingLeft` / `paddingRight` / `paddingTop` / `paddingBottom`
- `constraints`
- `fills` (array of Paint)
- `absoluteBoundingBox`

### TEXT

- `characters` — เนื้อความจริง
- `style` (TypeStyle): `fontFamily`, `fontSize`, `fontWeight`, `lineHeightPx`, `letterSpacing`, `textAlignHorizontal`
- `fills` — สีตัวอักษร

### COMPONENT / INSTANCE

- COMPONENT สืบทอด property ของ FRAME + `componentPropertyDefinitions`
- INSTANCE: `componentId` (ชี้ไป component ต้นทาง), `componentProperties`, `overrides`

### VECTOR / รูปทรง

- `strokes` (Paint array), `fills`, `strokeWeight`

### node types อื่น ๆ ที่ระบุไว้

RECTANGLE, ELLIPSE, POLYGON, BOOLEAN_OPERATION, TEXT_PATH, TABLE, CONNECTOR, STICKY, GROUP, SECTION, CANVAS

---

## 4. `GET /v1/images/:key` — render เป็นภาพ

อ้างอิง: https://developers.figma.com/docs/rest-api/file-endpoints/

| Param | ค่า |
|---|---|
| `ids` | **required** — node IDs ที่จะ render |
| `scale` | 0.01–4 |
| `format` | `jpg` \| `png` \| `svg` \| `pdf` |
| `svg_outline_text` | default true |
| `svg_include_id` | default false |
| `svg_include_node_id` | default false |
| `svg_simplify_stroke` | default true |
| `contents_only` | default true (ตัด content ที่ overlap ออก) |
| `use_absolute_bounds` | default false |
| `version` | version ID |

```json
{ "err": null, "images": { "1:23": "https://figma-alpha-api.s3.../abc.png" }, "status": 200 }
```

- **URL หมดอายุใน 30 วัน**
- ขนาด render สูงสุด **32 megapixels** — เกินกว่านั้นถูก scale ลงอัตโนมัติ

### `GET /v1/files/:key/images`

- คืน download URL ของ image fills ทั้งหมดในไฟล์ โดย key เป็น `imageRef`
- **URL หมดอายุภายใน 14 วัน**
- Tier 2

---

---

## 5. Authentication

อ้างอิง: https://developers.figma.com/docs/rest-api/authentication/

Figma มี 3 วิธี

### 5.1 Personal access token (PAT)

อ้างอิง: https://developers.figma.com/docs/rest-api/personal-access-tokens/

- สร้างที่ Figma → **Settings → Security → Generate new token**
- ตอนสร้างเลือก **expiration** และ **scopes** ได้
- ใช้ผ่าน header: `X-Figma-Token: <TOKEN>`
- เห็น token ครั้งเดียวตอนสร้าง — เอกสารเตือน "This will be your only chance to copy the token"
- ดู scope + last-used ของ token เดิมได้จาก Security tab, revoke ได้ทันที
- นิยาม: "gives the holder access to an account through the API **as if they were the user who generated the token**" → **สิทธิ์เท่ากับเจ้าของบัญชี** คือเข้าถึงไฟล์ทุกไฟล์ที่บัญชีนั้นเปิดได้ (รวมไฟล์ที่ถูกแชร์มาให้)
- เหมาะกับ "individual use, such as scripts or local tooling against your own Figma account"

### 5.2 OAuth 2 app

อ้างอิง: https://developers.figma.com/docs/rest-api/oauth-apps/

- Authorize URL: `https://www.figma.com/oauth?client_id=:client_id&redirect_uri=:callback&scope=:scope&state=:state&response_type=code`
- Token exchange: `POST https://api.figma.com/v1/oauth/token` (HTTP Basic Auth ด้วย base64 ของ `client_id:client_secret`)
- **authorization code หมดอายุใน 30 วินาที** — ต้องแลกทันที
- Response มี `access_token`, `refresh_token`, `expires_in` (**access token อายุ 90 วัน**), `user_id_string`
- Refresh: `POST https://api.figma.com/v1/oauth/refresh` — Figma เก็บ access token ได้แค่ 1 อันต่อ app ต่อ user ("Figma only maintains one access token per app for a user") → token เก่าใช้ไม่ได้หลัง refresh
- เรียก API ด้วย `Authorization: Bearer <TOKEN>`
- สร้าง app ที่ figma.com/developers/apps — ต้องผูกกับ team หรือ organization
- **Public app ต้องผ่าน Figma review; private app (ใช้ในทีม/org ตัวเอง) ไม่ต้อง review**
- รองรับ PKCE (แนะนำ, ใช้ `S256` เท่านั้น)

### 5.3 Plan access token

อ้างอิง: https://developers.figma.com/docs/rest-api/plan-access-tokens/

- ฟีเจอร์ของ **Organization / Enterprise plan เท่านั้น** — ไม่ผูกกับ user คนใดคนหนึ่ง
- เหมาะกับ CI/CD, logging, user-agnostic webhooks
- **ไม่น่าใช้ได้กับโครงงานนี้** ถ้าคณะไม่ได้ซื้อ Org/Enterprise

### 5.4 Scopes ที่เกี่ยวข้อง

อ้างอิง: https://developers.figma.com/docs/rest-api/scopes/

| Scope | ใช้ทำอะไร | หมายเหตุ |
|---|---|---|
| `file_content:read` | อ่านเนื้อไฟล์ (nodes, editor type) | **หัวใจของการตรวจงาน** |
| `file_metadata:read` | อ่าน metadata ของไฟล์ | |
| `file_comments:read` | อ่าน comments | |
| `file_comments:write` | โพสต์/ลบ comment และ reaction | **ใช้ส่ง feedback กลับได้** |
| `file_versions:read` | อ่าน version history ของไฟล์ที่เข้าถึงได้ | ตรวจ process/ความคืบหน้าได้ |
| `library_content:read` | อ่าน published components/styles ของไฟล์ | |
| `library_assets:read` | อ่านข้อมูล component/style ที่ published ทีละตัว | |
| `projects:read` | list projects และไฟล์ใน project | ใช้ดึงงานทั้งคลาสถ้าอยู่ใน team เดียวกัน |
| `project_metadata:read` | metadata ของ project | |
| `current_user:read` | ชื่อ อีเมล รูปโปรไฟล์ | ใช้ระบุตัว นศ. ตอน OAuth |
| `selections:read` | selection ล่าสุดในไฟล์ | ไม่จำเป็น |
| `files:read` | **deprecated** — สิทธิ์กว้าง | อย่าใช้ในของใหม่ |
| `file_variables:read` / `write` | variables | **Enterprise เท่านั้น** |
| `library_analytics:read` | design system analytics | **Enterprise เท่านั้น** |
| `org:*` | activity log / developer log / discovery / AI metering | **Enterprise + admin เท่านั้น** |

---

## 6. Rate limits — จุดที่ต้องระวังที่สุด

อ้างอิง: https://developers.figma.com/docs/rest-api/rate-limits/

- ใช้ **leaky bucket algorithm**; เกินแล้วได้ **HTTP 429**
- Response headers เมื่อโดนจำกัด: `Retry-After`, `X-Figma-Plan-Tier`, `X-Figma-Rate-Limit-Type`, `X-Figma-Upgrade-Link`
- **ชุดตัวเลขปัจจุบันมีผลตั้งแต่ 17 พฤศจิกายน 2025**
- Limit ขึ้นกับ 3 อย่าง: **seat type ของ user**, **tier ของ endpoint**, และ **plan ของ resource ที่ขอ**

### endpoint อยู่ tier ไหน

| Tier | Endpoints |
|---|---|
| **Tier 1** | GET file, GET file nodes, **GET image** |
| **Tier 2** | Comments, Dev Resources, Discovery, **GET image fills**, GET team projects, GET project files, GET local/published variables, Version History, Webhooks |
| **Tier 3** | Activity Logs, Components & Styles, Developer Logs, GET file metadata, Library Analytics, Payments, GET project metadata, Users, POST variables |

### ตัวเลข (requests per minute เว้นแต่ระบุ)

**Tier 1 — GET file / GET file nodes / GET image**

| Seat type | Starter | Professional | Organization | Enterprise |
|---|---|---|---|---|
| **View, Collab** | **สูงสุด 6 ต่อ *เดือน*** | สูงสุด 6/เดือน | สูงสุด 6/เดือน | สูงสุด 6/เดือน |
| **Dev, Full** | 10/min | 15/min | 20/min | 25/min |

**Tier 2**

| Seat type | Starter | Professional | Organization | Enterprise |
|---|---|---|---|---|
| View, Collab | 5/min | 5/min | 5/min | 5/min |
| Dev, Full | 25/min | 50/min | 100/min | 150/min |

**Tier 3**

| Seat type | Starter | Professional | Organization | Enterprise |
|---|---|---|---|---|
| View, Collab | 10/min | 10/min | 10/min | 10/min |
| Dev, Full | 50/min | 100/min | 150/min | 200/min |

> **ข้อควรระวัง:** ตัวเลขคอลัมน์ Enterprise ในตาราง Tier 1–3 ผมอ่านได้จากหน้าเดียวกันแต่ WebFetch สรุปมาไม่ตรงกัน 2 รอบ (25 vs 20 ใน Tier 1) — **ให้ถือว่าเลข Enterprise ยังไม่ verified 100%** ส่วนคอลัมน์ Starter/Professional ตรงกันทั้งสองรอบ ซึ่งเป็นคอลัมน์ที่เกี่ยวกับเราจริง
>
> เอกสารระบุเพิ่มว่า View/Collab tier 1 "the actual limit may be lower" ขึ้นกับ traffic

- **การนับ:** OAuth apps นับ per-user per-plan per-app; personal token นับ per-user per-plan
- **ไม่มี per-endpoint cost** — ขึ้นกับ tier/seat/plan เท่านั้น

### แปลว่าอะไรกับโครงงานนี้

- ถ้าบัญชีกลางของอาจารย์เป็น **Dev/Full seat บน Starter (ฟรี)** → **10 GET file/min** ⇒ ตรวจ 60 ไฟล์ใช้เวลาราว 6 นาที ถ้าเรียก 1 ครั้งต่อไฟล์ ซึ่ง**เพียงพอ**สำหรับคลาสขนาดปกติ ถ้า cache ผลไว้
- ถ้าใช้ **View/Collab seat → 6 ครั้งต่อเดือน = ใช้งานไม่ได้เลย** นี่เป็นความเสี่ยงที่ต้องเช็คก่อนเลือกสถาปัตยกรรม
- ควรออกแบบให้ **ดึงครั้งเดียวแล้วเก็บ JSON ลง DB** (snapshot ตอนส่งงาน) ไม่ใช่เรียก API ซ้ำทุกครั้งที่เปิดหน้าตรวจ

---

---

## 7. Type definitions ที่แม่นยำ (จาก official OpenAPI spec)

Figma เผยแพร่ spec อย่างเป็นทางการที่ repo `figma/rest-api-spec` — ใช้ยืนยันชื่อ field ได้แน่นอน
อ้างอิง: https://raw.githubusercontent.com/figma/rest-api-spec/main/dist/api_types.ts

### `RGBA` — ค่าสี

```ts
type RGBA = { r: number; g: number; b: number; a: number }
```

> **สำคัญมากสำหรับการคำนวณ contrast:** `r/g/b/a` เป็น float **0–1** ไม่ใช่ 0–255 ต้องคูณ 255 ก่อนแปลงเป็น sRGB

### `SolidPaint` / `BasePaint`

```ts
type BasePaint  = { visible?: boolean; opacity?: number; blendMode: BlendMode }
type SolidPaint = { type: 'SOLID'; color: RGBA; boundVariables?: { color?: VariableAlias } } & BasePaint
```

Paint มีชนิดอื่นด้วย (gradient, image) — ถ้าเจอ `type` ที่ไม่ใช่ `SOLID` จะคำนวณ contrast ตรง ๆ ไม่ได้

### `BaseTypeStyle` (typography — หัวใจของการวัด type scale)

```ts
type BaseTypeStyle = {
  fontFamily?: string
  fontPostScriptName?: string | null
  fontStyle?: string
  italic?: boolean
  fontWeight?: number
  fontSize?: number
  textCase?: 'ORIGINAL'|'UPPER'|'LOWER'|'TITLE'|'SMALL_CAPS'|'SMALL_CAPS_FORCED'
  textAlignHorizontal?: 'LEFT'|'RIGHT'|'CENTER'|'JUSTIFIED'
  textAlignVertical?: 'TOP'|'CENTER'|'BOTTOM'
  letterSpacing?: number
  fills?: Paint[]
  hyperlink?: Hyperlink
  opentypeFlags?: { [key: string]: number }
  semanticWeight?: 'BOLD'|'NORMAL'
  semanticItalic?: 'ITALIC'|'NORMAL'
}
```

### `TypeStyle` (= BaseTypeStyle + ส่วนต่อไปนี้)

```ts
type TypeStyle = {
  paragraphSpacing?: number
  paragraphIndent?: number
  listSpacing?: number
  textDecoration?: 'NONE'|'STRIKETHROUGH'|'UNDERLINE'
  textAutoResize?: 'NONE'|'WIDTH_AND_HEIGHT'|'HEIGHT'|'TRUNCATE'
  textTruncation?: 'DISABLED'|'ENDING'
  maxLines?: number
  lineHeightPx?: number
  lineHeightPercent?: number
  lineHeightPercentFontSize?: number
  lineHeightUnit?: 'PIXELS'|'FONT_SIZE_%'|'INTRINSIC_%'
  isOverrideOverTextStyle?: boolean
  boundVariables?: { fontFamily?; fontSize?; fontStyle?; fontWeight?;
                     letterSpacing?; lineHeight?; paragraphSpacing?; paragraphIndent? }
} & BaseTypeStyle
```

### `HasLayoutTrait` — เรขาคณิต + auto-layout ของลูก

```ts
type HasLayoutTrait = {
  absoluteBoundingBox: Rectangle | null
  absoluteRenderBounds: Rectangle | null
  preserveRatio?: boolean
  constraints?: LayoutConstraint
  relativeTransform?: Transform
  size?: Vector
  layoutAlign?: 'INHERIT'|'STRETCH'|'MIN'|'CENTER'|'MAX'
  layoutGrow?: 0 | 1
  layoutPositioning?: 'AUTO'|'ABSOLUTE'
  minWidth?; maxWidth?; minHeight?; maxHeight?: number
  layoutSizingHorizontal?: 'FIXED'|'HUG'|'FILL'
  layoutSizingVertical?: 'FIXED'|'HUG'|'FILL'
  gridRowCount?; gridColumnCount?; gridRowGap?; gridColumnGap?: number
  // ... grid child alignment/span fields
}

type Rectangle = { x: number; y: number; width: number; height: number }
type LayoutConstraint = {
  vertical: 'TOP'|'BOTTOM'|'CENTER'|'TOP_BOTTOM'|'SCALE'
  horizontal: 'LEFT'|'RIGHT'|'CENTER'|'LEFT_RIGHT'|'SCALE'
}
```

### `Component` และ `Style` (ค่าใน map ระดับไฟล์)

```ts
type Component = {
  key: string
  name: string
  description: string
  componentSetId?: string
  documentationLinks: DocumentationLink[]
  remote: boolean          // true = มาจาก library ภายนอก
}

type Style = {
  key: string
  name: string
  description: string
  remote: boolean
  styleType: StyleType     // FILL | TEXT | EFFECT | GRID
}
```

> `remote: boolean` มีค่ามาก — แยกได้ว่า นศ. สร้าง design system เองหรือหยิบมาจาก community library

### ตัวอย่าง TEXT node จริง (ย่อ แต่ field ถูกต้องตาม spec)

```json
{
  "id": "12:345",
  "name": "Heading / Page title",
  "type": "TEXT",
  "visible": true,
  "absoluteBoundingBox": { "x": 24, "y": 88, "width": 327, "height": 34 },
  "constraints": { "vertical": "TOP", "horizontal": "LEFT" },
  "layoutSizingHorizontal": "FILL",
  "characters": "สั่งอาหารใกล้คุณ",
  "style": {
    "fontFamily": "Inter",
    "fontPostScriptName": "Inter-SemiBold",
    "fontWeight": 600,
    "fontSize": 24,
    "textAlignHorizontal": "LEFT",
    "textAlignVertical": "TOP",
    "letterSpacing": -0.5,
    "lineHeightPx": 33.6,
    "lineHeightPercent": 100,
    "lineHeightUnit": "PIXELS"
  },
  "fills": [
    { "type": "SOLID", "blendMode": "NORMAL", "visible": true, "opacity": 1,
      "color": { "r": 0.101, "g": 0.101, "b": 0.117, "a": 1 } }
  ],
  "styles": { "text": "S:abcdef123...", "fill": "S:987654..." }
}
```

> `styles` บน node เป็น map `{ fill | text | effect | grid | stroke : styleId }` ที่ชี้ไป key ใน `styles` ระดับไฟล์ — **ถ้า node ไม่มี key นี้ แปลว่าใส่สี/ฟอนต์แบบ hard-code ไม่ได้ใช้ named style** ซึ่งเป็นสัญญาณคุณภาพโดยตรง
> (โครงสร้าง `styles` บน node สังเกตได้จาก field `styles` ที่ปรากฏใน response ระดับไฟล์และ node — ชื่อ key ย่อยระดับ node **ยังไม่ verified จากหน้า docs โดยตรงในรอบค้นนี้** ให้ทีมยืนยันตอน spike จริง)

### ตัวอย่าง FRAME node ที่ใช้ auto-layout

```json
{
  "id": "12:300",
  "name": "Card / Restaurant",
  "type": "FRAME",
  "layoutMode": "VERTICAL",
  "itemSpacing": 8,
  "paddingLeft": 16, "paddingRight": 16, "paddingTop": 12, "paddingBottom": 12,
  "primaryAxisAlignItems": "MIN",
  "counterAxisAlignItems": "STRETCH",
  "absoluteBoundingBox": { "x": 24, "y": 140, "width": 327, "height": 220 },
  "cornerRadius": 12,
  "fills": [ { "type": "SOLID", "blendMode": "NORMAL",
               "color": { "r": 1, "g": 1, "b": 1, "a": 1 } } ],
  "effects": [ { "type": "DROP_SHADOW", "visible": true,
                 "color": { "r":0,"g":0,"b":0,"a":0.08 },
                 "offset": {"x":0,"y":2}, "radius": 8 } ],
  "children": []
}
```

---

## 8. Prototype / flow — ดึงได้จริง

อ้างอิง: https://developers.figma.com/docs/rest-api/file-node-types/

### บน CANVAS node (= 1 page)

| Property | คำอธิบายจาก docs |
|---|---|
| `flowStartingPoints` | "A array of flow starting points sorted by its position in the prototype settings panel." |
| `prototypeStartNodeID` | "Node ID that corresponds to the start frame for prototypes. **This is deprecated with the introduction of multiple flows.**" |
| `prototypeDevice` | "The device used to view a prototype." |
| `children`, `backgroundColor`, `exportSettings`, `measurements` | — |

### บน FRAME node

| Property | คำอธิบายจาก docs |
|---|---|
| `interactions` | "List of prototype interactions on this node, which includes both **the method of interaction** with this node in a prototype, and **the behavior of that interaction**." |
| `transitionNodeID` | "Node ID of node to transition to in prototyping." |
| `transitionDuration` | "The duration of the prototyping transition on this node (in milliseconds)." |
| `transitionEasing` | "The easing curve used in the prototyping transition on this node." |
| `overflowDirection` | "Defines the scrolling behavior of the frame, if there exist contents outside of the frame boundaries." |

**สรุปข้อ 3 ของ ticket: ดึง prototype/flow ได้** — สร้างกราฟ navigation จาก `interactions` / `transitionNodeID` ได้เต็มรูปแบบ และรู้จุดเริ่ม flow จาก `flowStartingPoints` ⇒ rubric ข้อ user flow ตรวจอัตโนมัติได้จริง

> ยังไม่ได้ verify โครงสร้างภายในของ `interactions` (Reaction / Trigger / Action) จากหน้า REST API โดยตรงในรอบนี้ — Plugin API มีหน้า `Reaction` / `Action` อธิบายไว้ (https://developers.figma.com/docs/plugins/api/Reaction/) แต่ REST อาจ serialize ต่างกัน ให้ทีม dump JSON จริงตอน spike

---

## 9. Comments API — ส่ง feedback กลับเข้าไฟล์ นศ. ได้

อ้างอิง: https://developers.figma.com/docs/rest-api/comments-endpoints/ และ https://developers.figma.com/docs/rest-api/comments-types/

| Endpoint | Scope | Tier |
|---|---|---|
| `GET /v1/files/:key/comments` (query `as_md`) | `file_comments:read` | 2 |
| `POST /v1/files/:file_key/comments` | `file_comments:write` | 2 |
| `DELETE /v1/files/:file_key/comments/:comment_id` (ลบได้เฉพาะ comment ที่ตัวเองสร้าง) | `file_comments:write` | 2 |
| `GET/POST/DELETE /v1/files/:file_key/comments/:comment_id/reactions` | read / write | 2 |

### POST body

- `message` (string, **required**)
- `comment_id` (string, optional) — ใส่เพื่อตอบ (reply) ใต้ comment เดิม
- `client_meta` (optional) — ตำแหน่ง: `Vector` | `FrameOffset` | `Region` | `FrameOffsetRegion`

```ts
type FrameOffset = { node_id: string; node_offset: Vector }   // Vector = {x, y}
```

⇒ **ปักหมุด comment ลงตรงจุดใน frame ที่มีปัญหาได้** เช่น ชี้ไปที่ปุ่มที่ contrast ไม่ผ่าน WCAG

### Comment object

```json
{
  "id": "1234567890",
  "file_key": "AbCdEf123",
  "parent_id": "",
  "user": { "id": "…", "handle": "AI Grader", "img_url": "…", "email": "…" },
  "created_at": "2026-08-05T04:11:00Z",
  "resolved_at": null,
  "message": "ปุ่ม 'สั่งเลย' contrast 2.9:1 ไม่ผ่าน WCAG AA (ต้อง ≥ 4.5:1)",
  "client_meta": { "node_id": "12:345", "node_offset": { "x": 12, "y": 8 } },
  "order_id": "42",
  "reactions": []
}
```

Errors: 403 (token ไม่ถูกต้อง), 404 (ไม่พบไฟล์)

> ข้อควรคิด: comment จะไปโผล่ในไฟล์ นศ. ในนามบัญชีที่ถือ token — ถ้าใช้บัญชีกลาง "AI Grader" นศ. จะเห็นว่าใครคอมเมนต์ ต้องคุยเรื่อง UX/นโยบายว่าจะให้ AI คอมเมนต์ตรงหรือให้อาจารย์อนุมัติก่อน

---

## 10. Components & Styles endpoints (team library)

อ้างอิง: https://developers.figma.com/docs/rest-api/component-types/

Object ที่ endpoint กลุ่มนี้คืน (`GET /v1/files/:key/components`, `/component_sets`, `/styles`, และเวอร์ชัน team) มี field:

- Component / ComponentSet: `key`, `file_key`, `node_id`, `name`, `description`, `thumbnail_url`, `created_at`, `updated_at`, `user`, `containing_frame`
- Style: เพิ่ม `style_type` = `FILL` | `TEXT` | `EFFECT` | `GRID`
- `FrameInfo` (ค่าใน `containing_frame`): `node_id`, `name`, `backgroundColor`, `pageId`, `pageName`, `containingComponentSet`

> **ข้อจำกัดสำคัญ:** endpoint กลุ่มนี้เป็นของ **team library assets** (ของที่ published แล้ว) ไม่ใช่ component ที่อยู่ในไฟล์เฉย ๆ
> ⇒ สำหรับงาน นศ. ที่มักไม่ได้ publish library ให้ **นับ component จาก `components` / `componentSets` map ใน response ของ `GET /v1/files/:key` แทน** ซึ่งใช้ได้เสมอ

---

---

## 11. Endpoint เสริมที่มีประโยชน์

### `GET /v1/files/:key/meta` — เช็คไฟล์แบบถูก ๆ ก่อนดึงของหนัก

อ้างอิง: https://developers.figma.com/docs/rest-api/file-endpoints/

- Scope `file_metadata:read`, **Tier 3** (โควตาสูงกว่า Tier 1 มาก)
- คืน `name`, `lastModified`, `thumbnailUrl`, `editorType`, `linkAccess`, `version` — **ไม่มีเนื้อไฟล์**
- `linkAccess` = ระดับสิทธิ์ของ share link มี 5 ค่า: `inherit`, `view`, `edit`, `org_view`, `org_edit`
  - `inherit` = ค่า default ของไฟล์ที่สร้างใน team project (สืบทอดสิทธิ์ของ project)
  - `org_view` / `org_edit` = จำกัดเฉพาะคนใน org

> **แนวปฏิบัติ:** ตอน นศ. วางลิงก์ในระบบ ให้ยิง `/meta` ก่อน (Tier 3 โควตาเยอะ) เพื่อ validate ว่าลิงก์ถูก + เข้าถึงได้ + ดู `linkAccess` แล้วแจ้ง นศ. ทันทีถ้าตั้งสิทธิ์ผิด **โดยไม่เปลือง quota Tier 1**

### `GET /v1/files/:key/versions`

อ้างอิง: https://developers.figma.com/docs/rest-api/version-history-endpoints/

- Scope `file_versions:read`, **Tier 2**
- คืนรายการ version เรียงตามเวลาที่สร้าง (มี pagination `next_page`/`prev_page`)
- แต่ละ version มี who / when / title / description
- ⇒ ใช้ประเมิน **กระบวนการทำงาน** ได้ (ทำสม่ำเสมอ หรือทำรวดเดียวคืนก่อนส่ง) และตรวจว่าไฟล์ถูกแก้หลังกำหนดส่งหรือไม่

### Webhooks V2

อ้างอิง: https://developers.figma.com/docs/rest-api/webhooks/ — Tier 2 (scope `webhooks:read` / `webhooks:write`) แจ้งเตือนเมื่อไฟล์เปลี่ยน; ใช้ตรวจการแก้ไขหลังส่งได้ แต่ปกติ webhook ผูกกับ **team** ที่เราคุม จึงใช้ได้ต่อเมื่องาน นศ. อยู่ใน team ของวิชา

---

## 12. Plan / seat / education — ข้อจำกัดจริง

### Seat types

อ้างอิง: https://help.figma.com/hc/en-us/articles/360039960434-Manage-seats-in-Figma

| Seat | ได้อะไร |
|---|---|
| **Full** | เข้าถึงเต็มทุกผลิตภัณฑ์ (Figma Design, Dev Mode, FigJam, Slides ฯลฯ) |
| **Dev** | Dev Mode เต็ม + FigJam/Slides/Buzz; ใน Figma Design ได้แค่ **view และ comment** |
| **Collab** | FigJam/Slides/Buzz เต็ม; Figma Design ได้แค่ view/comment; Dev Mode แบบ basic inspection |
| **View** | **ฟรี ไม่ต้องซื้อ** |

**เชื่อมกับ rate limit:** Tier 1 ให้ **Dev และ Full seat** เท่านั้นที่ได้โควตาต่อนาที; **View และ Collab ได้แค่ ~6 ครั้งต่อเดือน**
⇒ **บัญชีที่ระบบใช้ยิง API ต้องเป็น Dev หรือ Full seat** ไม่งั้นใช้งานไม่ได้เลย

### Figma for Education

อ้างอิง: https://help.figma.com/hc/en-us/articles/360041061214-Figma-for-Education และ https://www.figma.com/education/

- **นักศึกษาและอาจารย์ระดับอุดมศึกษา (higher education) ได้ Professional plan ฟรี**
- (ระดับมัธยม/K-12 ได้ Enterprise plan)
- ต้องสมัครด้วยอีเมลของสถาบัน, ไม่ได้เปิดทุกประเทศ
- อายุ **1 ปี** สำหรับ higher education แล้วต้องสมัครใหม่ (bootcamp 6 เดือน / ครู bootcamp 2 ปี)
- หน้า help **ไม่ได้พูดถึง API เลย** → ถือว่าได้สิทธิ์เท่า Professional โดยปริยาย แต่ **ยังไม่ verified โดยตรงจากเอกสาร**

⇒ ถ้าอาจารย์ผู้สอนมีบัญชี education (Professional) และเป็น Full seat → **Tier 1 = 15 req/min** ซึ่งเพียงพอกับการตรวจงานทั้งคลาสแบบ batch

### ราคา

- REST API **ไม่มีค่าใช้จ่ายแยก** — ไม่พบหน้าใดใน developers.figma.com ที่คิดเงินต่อ request; ต้นทุนคือค่า seat/plan ตามปกติ (https://www.figma.com/pricing/)
- ไม่พบหลักฐานว่า education plan ถูกจำกัด API เป็นพิเศษ — **unverified**

---

## 13. การเข้าถึงไฟล์ของนักศึกษา — ตัวเลือกและข้อดีข้อเสีย

### ข้อเท็จจริงที่ยืนยันได้

- PAT ให้สิทธิ์ **เท่ากับเจ้าของบัญชี**: "Personal access tokens allow third-party programs to access **all of your files and data** in Figma" และ "**It's not possible to restrict access when using a personal access token**"
  (https://help.figma.com/hc/en-us/articles/8085703771159-Manage-personal-access-tokens)
  ⇒ PAT อ่านได้ทุกไฟล์ที่บัญชีนั้นเปิดได้ = ไฟล์ที่ตัวเองสร้าง + ไฟล์ที่ถูกแชร์มา + ไฟล์ใน team/project ที่เป็นสมาชิก
- OAuth ให้ app "act on behalf of individual Figma users" โดยผู้ใช้กดยินยอมและเลือก scope ได้
  (https://developers.figma.com/docs/rest-api/oauth-apps/)
- ไฟล์ที่ตั้ง **password protection เข้าถึงผ่าน REST API ไม่ได้** — ไม่มี param/header สำหรับส่ง password
  (https://forum.figma.com/ask-the-community-7/rest-api-how-can-i-access-a-password-protected-file-using-a-figma-api-by-passing-password-49020)
- ไม่พบสิทธิ์ = ได้ **403 Forbidden**; ไฟล์ไม่มีจริง/เข้าไม่ถึง = **404**
  (https://developers.figma.com/docs/rest-api/errors/)

### ⚠️ จุดที่ยัง UNVERIFIED และต้อง spike ก่อนตัดสินใจสถาปัตยกรรม

**คำถาม: ไฟล์ที่ตั้ง "Anyone with the link → can view" อ่านผ่าน REST API ด้วย PAT ของคนที่ไม่ได้ถูกเชิญ ได้หรือไม่?**

- **เอกสารทางการของ Figma ไม่ได้ระบุเรื่องนี้ไว้ตรง ๆ** ทั้งในหน้า authentication, personal access tokens และ guide to sharing and permissions
- แหล่งชุมชนหลายที่ระบุว่า **ต้องเชิญบัญชีที่ถือ token เข้าไฟล์ก่อน** จึงจะเรียก API ได้ (workaround ที่แนะนำกันคือ invite อีเมลของเจ้าของ PAT เข้าไฟล์)
- **ผลของคำถามนี้เปลี่ยนสถาปัตยกรรมทั้งหมด** → **ให้ทีมทำ spike 30 นาที**: สร้างไฟล์ในบัญชี A ตั้ง link เป็น "anyone with the link can view" แล้วยิง `GET /v1/files/:key` ด้วย PAT ของบัญชี B ที่ไม่ได้ถูกเชิญ ดูว่าได้ 200 หรือ 403
- **จนกว่าจะพิสูจน์ได้ ให้ออกแบบโดยสมมติว่าต้องมีสิทธิ์ชัดเจน (ทางเลือก B/C ด้านล่าง)**

### เปรียบเทียบ 4 ทางเลือก

| # | วิธี | ข้อดี | ข้อเสีย / ความเสี่ยง |
|---|---|---|---|
| **A** | **นศ. แชร์ public view link** แล้วระบบใช้ PAT บัญชีกลางอ่าน | ง่ายที่สุดสำหรับ นศ. ไม่ต้อง login อะไรเพิ่ม; ระบบใช้ token เดียว | **ยังไม่ยืนยันว่าทำได้จริง** (ดูด้านบน); ถ้าลิงก์รั่ว ใครก็เห็นงาน; นศ. ตั้งค่าผิดง่าย (ลืมเปิด public → 403/404) |
| **B** | **นศ. เชิญบัญชีกลางของวิชา** (เช่น `ux-grader@kmitl.ac.th`) เป็น viewer ในไฟล์ | ยืนยันได้แน่นอนว่า API เข้าถึงได้; PAT ตัวเดียวจบ; **โควตาผูกกับบัญชีเดียว จัดการง่าย** | ต้องสอน นศ. ให้ invite (มี friction 1 ขั้น); ถ้าเชิญเป็น "can edit" จะเสี่ยงเกินจำเป็น — ต้องกำหนดให้เป็น view เท่านั้น |
| **C** | **สร้าง Figma team ของวิชา** แล้วให้ นศ. ทำงานใน project ของ team นั้น | ระบบเห็นทุกไฟล์อัตโนมัติผ่าน `projects:read` + `GET /v1/projects/:id/files`; ใช้ **webhook** ตรวจการแก้ไขได้; ไม่ต้องเก็บลิงก์ทีละคน | นศ. ต้องเข้าร่วม team ตั้งแต่ต้นเทอม; ไฟล์ถือว่าอยู่ใน team ของวิชา ต้องเคลียร์เรื่องความเป็นเจ้าของงาน; อาจต้องจ่ายค่า seat ถ้าให้ นศ. edit ใน team ที่เป็น Professional (การให้สิทธิ์ edit = ต้องมี Full seat) |
| **D** | **นศ. ทำ OAuth ให้ระบบ** (private OAuth app ของคณะ) | นศ. ควบคุมสิทธิ์เอง ถอนได้; ระบบเข้าถึงเฉพาะ scope ที่ขอ; **private app ไม่ต้องผ่าน Figma review** | ซับซ้อนที่สุด (ต้องทำ OAuth flow + เก็บ refresh token); **access token อายุ 90 วัน** ต้องมี refresh job; **rate limit นับ per-user** ⇒ ข้อดีซ่อนอยู่: โควตากระจายไปตาม นศ. แต่ละคน ไม่กระจุกที่บัญชีเดียว; ถ้า นศ. เป็น View/Collab seat จะติดเพดาน 6 ครั้ง/เดือน |

### ข้อเสนอ (สำหรับเอกสารส่งมอบ)

**MVP ใช้ทางเลือก B** (เชิญบัญชีกลางเป็น viewer) เพราะ

1. ยืนยันได้แน่นอนตามเอกสาร ไม่ต้องพึ่งพฤติกรรมที่ยัง unverified
2. ระบบถือ PAT เดียว — โครงสร้างเรียบง่าย เหมาะกับทีม 2-3 คน
3. บัญชีกลางถ้าใช้บัญชี education ของอาจารย์ (Professional + Full seat) → Tier 1 = 15/min เพียงพอ
4. รองรับ Comments API ได้ทันที (บัญชีกลางโพสต์ comment กลับได้ ถ้าได้สิทธิ์ comment)

**เตรียม C ไว้เป็น upgrade path** ถ้าคณะยอมตั้ง Figma team ของวิชา จะได้ webhook + ดึงไฟล์อัตโนมัติ
**เลื่อน D ออกไปหลัง MVP** — ต้นทุนวิศวกรรมสูงเกินสำหรับปีแรก

**ต้องมี validation ตอนรับลิงก์เสมอ:** parse `file_key` จาก URL รูปแบบ `https://www.figma.com/:file_type/:file_key/:file_name` → ยิง `/meta` → ถ้า 403/404 แสดงข้อความสอน นศ. ให้แก้สิทธิ์ทันที ไม่ใช่ปล่อยให้ fail ตอนตรวจ

---

## 14. ⭐ สัญญาณคุณภาพ UX/UI ที่คำนวณได้แบบ deterministic (ไม่ต้องใช้ LLM)

**นี่คือผลลัพธ์สำคัญที่สุดของ ticket นี้** — ทุกข้อคำนวณจาก JSON ของ `GET /v1/files/:key` ล้วน ๆ ผลลัพธ์ทำซ้ำได้ 100% อธิบายได้ และอุทธรณ์ได้ ต่างจากคะแนนที่ LLM ให้

### A. Accessibility / Contrast

| # | สัญญาณ | คำนวณจาก | หมายเหตุ |
|---|---|---|---|
| A1 | **WCAG contrast ratio ของข้อความ** | `TEXT.fills[].color` (RGBA 0–1) เทียบกับ `fills` ของ FRAME/parent ที่อยู่หลัง (หาโดย hit-test ด้วย `absoluteBoundingBox`) → แปลงเป็น sRGB → relative luminance → `(L1+0.05)/(L2+0.05)` | **ทำได้จริงและแม่นที่สุด** ต้องระวัง: `opacity` ของ paint, `blendMode`, และพื้นหลังที่เป็น gradient/image (กรณีนั้นให้ flag ว่า "ตรวจไม่ได้" แทนการเดา) |
| A2 | **ผ่าน WCAG AA / AAA หรือไม่** | A1 + `fontSize` และ `fontWeight` (large text = ≥18pt หรือ ≥14pt bold → เกณฑ์ 3:1 แทน 4.5:1) | ได้ `fontSize`/`fontWeight` ตรง ๆ จาก TypeStyle |
| A3 | **ขนาดตัวอักษรเล็กเกินไป** | นับ TEXT node ที่ `style.fontSize` < 12 (หรือเกณฑ์ที่ rubric กำหนด) | |
| A4 | **ขนาด touch target** | `absoluteBoundingBox.width/height` ของ node ที่มี `interactions` หรือชื่อ layer สื่อว่าเป็นปุ่ม | เกณฑ์ 44×44pt (iOS) / 48×48dp (Android) |
| A5 | **สื่อความหมายด้วยสีอย่างเดียว** | บางส่วน — ตรวจได้ว่า node ที่มีสีต่างกันมี TEXT/ICON กำกับหรือไม่ | heuristic ไม่ใช่ข้อสรุป ควรส่งต่อให้ LLM/มนุษย์ |

### B. Typography / Type scale

| # | สัญญาณ | คำนวณจาก |
|---|---|---|
| B1 | **จำนวน `fontSize` ที่ไม่ซ้ำกันทั้งไฟล์** | `distinct(TEXT.style.fontSize)` — ยิ่งเยอะยิ่งไม่มีระบบ (เช่น >8 ค่า = สัญญาณแย่) |
| B2 | **จำนวน `fontFamily` ที่ไม่ซ้ำ** | `distinct(TEXT.style.fontFamily)` — เกิน 2-3 ตระกูล = ไม่สม่ำเสมอ |
| B3 | **จำนวน `fontWeight` ที่ไม่ซ้ำ** | `distinct(TEXT.style.fontWeight)` |
| B4 | **type scale เป็นอัตราส่วนหรือไม่** | เรียง fontSize แล้วดูอัตราส่วนระหว่างขั้น — ใกล้ค่าคงที่ (1.125/1.2/1.25/1.333/1.5) = มี modular scale |
| B5 | **line-height ที่กำหนดไว้ vs ปล่อย auto** | `style.lineHeightUnit` / `lineHeightPx` / `lineHeightPercent`; อัตราส่วน `lineHeightPx / fontSize` ควรอยู่ ~1.4–1.6 สำหรับ body |
| B6 | **ความยาวบรรทัด (line length)** | `absoluteBoundingBox.width` ของ TEXT ÷ ประมาณความกว้างตัวอักษรจาก `fontSize` → เตือนถ้าเกิน ~75 ตัวอักษร |
| B7 | **ใช้ text style ที่ตั้งชื่อไว้กี่ %** | นับ TEXT node ที่มี key `text` ใน `styles` ของ node เทียบกับ TEXT ทั้งหมด |
| B8 | **letterSpacing / textCase ที่ผิดปกติ** | `style.letterSpacing`, `style.textCase` (เช่น `UPPER` ทั้งย่อหน้ายาว = อ่านยาก) |

### C. Color / Design tokens

| # | สัญญาณ | คำนวณจาก |
|---|---|---|
| C1 | **จำนวนสีที่ไม่ซ้ำทั้งไฟล์** | รวบรวมทุก `SolidPaint.color` จาก `fills` + `strokes` แล้ว dedupe — palette บาน = ไม่มีระบบสี |
| C2 | **ใช้ fill style ที่ตั้งชื่อไว้กี่ %** | นับ node ที่มี key `fill` ใน `styles` ของ node |
| C3 | **จำนวน named styles ทั้งไฟล์ แยกตาม `styleType`** | `styles` map ระดับไฟล์ → นับตาม `FILL` / `TEXT` / `EFFECT` / `GRID` |
| C4 | **design system เป็นของตัวเองหรือหยิบมา** | `Style.remote` / `Component.remote` — `true` = มาจาก library ภายนอก |
| C5 | **ใช้ Variables (design tokens) หรือไม่** | `boundVariables` บน node และใน TypeStyle | ⚠️ อ่าน variables ผ่าน endpoint ต้องเป็น **Enterprise** (`file_variables:read`) แต่ field `boundVariables` ที่ฝังมาใน file response น่าจะเห็นได้ — **unverified** |

### D. Layout / Spacing consistency

| # | สัญญาณ | คำนวณจาก |
|---|---|---|
| D1 | **ใช้ auto-layout กี่ % ของ frame** | นับ FRAME ที่ `layoutMode !== 'NONE'` เทียบกับ FRAME ทั้งหมด — ตัวชี้วัดวุฒิภาวะทางเทคนิคที่ตรงที่สุดตัวหนึ่ง |
| D2 | **spacing เกาะ grid 8pt หรือไม่** | ตรวจว่า `itemSpacing`, `paddingLeft/Right/Top/Bottom` หารด้วย 8 (หรือ 4) ลงตัวกี่ % |
| D3 | **จำนวนค่า spacing ที่ไม่ซ้ำ** | `distinct(itemSpacing ∪ padding*)` — ยิ่งเยอะยิ่งมั่ว |
| D4 | **การจัดแนว (alignment)** | `absoluteBoundingBox.x` ของ element ในหน้าเดียวกัน — นับจำนวน "แนว x" ที่ไม่ซ้ำ; element ที่คลาดจากแนวไป 1-3px = ไม่ได้ตั้งใจ ตรวจจับได้แม่น |
| D5 | **margin ขอบจอสม่ำเสมอ** | ระยะจาก `absoluteBoundingBox` ของลูกถึงขอบ frame หน้าจอ |
| D6 | **element ล้นออกนอก frame** | เทียบ `absoluteBoundingBox` ลูกกับพ่อ + ดู `overflowDirection` |
| D7 | **element ทับซ้อนกันโดยไม่ตั้งใจ** | ตรวจ intersection ของ `absoluteBoundingBox` ระหว่าง sibling ที่ไม่ควรทับ |
| D8 | **ใช้ layout grid หรือไม่** | มี style `styleType: GRID` / `layoutGrids` บน frame |
| D9 | **ใช้ constraints หรือปล่อย default** | `constraints.vertical/horizontal` — ถ้าเป็น `TOP`/`LEFT` ทุกอันแปลว่าไม่ได้คิดเรื่อง responsive |
| D10 | **ใช้ layoutSizing (HUG/FILL) หรือ FIXED ล้วน** | `layoutSizingHorizontal/Vertical` |

### E. Componentization / ระบบ

| # | สัญญาณ | คำนวณจาก |
|---|---|---|
| E1 | **จำนวน COMPONENT และ COMPONENT_SET** | `components` / `componentSets` map ระดับไฟล์ |
| E2 | **อัตราส่วน INSTANCE : node ทั้งหมด** | นับ node `type === 'INSTANCE'` — สูง = นำกลับมาใช้ซ้ำจริง |
| E3 | **component ที่สร้างแล้วไม่ถูกใช้เลย** | component ที่ไม่มี INSTANCE ไหนมี `componentId` ชี้มา |
| E4 | **INSTANCE ที่ถูก override เยอะผิดปกติ** | `overrides` array ยาว = ใช้ component ผิดวิธี |
| E5 | **มี component variants หรือไม่** | `componentSetId` บน Component, `componentPropertyDefinitions` บน COMPONENT |
| E6 | **การตั้งชื่อ layer** | `name` ของ node — นับ % ที่ยังเป็นชื่อ default (`Frame 123`, `Rectangle 4`, `Group 7`) = สัญญาณความเรียบร้อยที่ตรวจง่ายและเถียงไม่ได้ |
| E7 | **โครงสร้าง layer ลึกเกินไป** | ความลึกสูงสุดของ tree; GROUP ซ้อน GROUP ที่ไม่จำเป็น |
| E8 | **มี layer ที่ซ่อนไว้ค้าง** | `visible === false` |

### F. ความครบถ้วนของงานส่ง

| # | สัญญาณ | คำนวณจาก |
|---|---|---|
| F1 | **จำนวนหน้าจอ (screens)** | นับ FRAME ระดับบนสุดใต้ CANVAS ที่ขนาดตรงกับ device preset |
| F2 | **จำนวน page ใน Figma file** | นับ CANVAS node |
| F3 | **ขนาด artboard สอดคล้องกับ device จริงไหม** | `absoluteBoundingBox.width/height` เทียบ preset (390×844 iPhone, 1440×1024 desktop ฯลฯ) + `prototypeDevice` |
| F4 | **มี state ครบไหม (empty/loading/error)** | keyword match บน `name` ของ frame — heuristic |

### G. Prototype / User flow

| # | สัญญาณ | คำนวณจาก |
|---|---|---|
| G1 | **มี prototype หรือเป็นภาพนิ่งเฉย ๆ** | มี `interactions` / `transitionNodeID` ที่ไหนบ้างหรือไม่ |
| G2 | **จำนวน flow และจุดเริ่มต้น** | `flowStartingPoints` บน CANVAS |
| G3 | **หน้าจอที่ไม่มีทางเข้าถึง (orphan screens)** | สร้าง directed graph จาก `interactions`/`transitionNodeID` → หา node ที่ไม่มี incoming edge และไม่ใช่ flow starting point |
| G4 | **หน้าจอตัน (dead ends) — เข้าได้แต่ออกไม่ได้** | node ที่ไม่มี outgoing edge |
| G5 | **ความลึกของ flow / จำนวนคลิกถึงเป้าหมาย** | shortest path บนกราฟจาก starting point |
| G6 | **มีทางกลับ (back navigation) ไหม** | ตรวจ edge ย้อนกลับ |
| G7 | **ความสม่ำเสมอของ transition** | `transitionDuration`, `transitionEasing` — ค่ากระจัดกระจาย = ไม่ได้ตั้งใจออกแบบ |

### H. Process / ความซื่อสัตย์

| # | สัญญาณ | คำนวณจาก |
|---|---|---|
| H1 | **รูปแบบการทำงานตลอดเทอม** | `GET /v1/files/:key/versions` — กระจายตัวของ timestamp |
| H2 | **แก้ไขหลังกำหนดส่ง** | `lastModified` เทียบ deadline |
| H3 | **สัดส่วนของที่หยิบมาจาก library ภายนอก** | `remote: true` ใน `components`/`styles` — เยอะมาก = อาจไม่ได้ออกแบบเอง |

### สิ่งที่ **ทำ deterministic ไม่ได้** (ต้องพึ่ง LLM หรือมนุษย์)

- คุณภาพของ **information architecture** และการจัดลำดับความสำคัญของเนื้อหา
- ความเหมาะสมของ **copy / microcopy** ภาษาไทย
- ความสอดคล้องกับ **โจทย์/persona/user need** ที่กำหนดในวิชา
- ความสวยงามเชิงสุนทรียะ, ความเหมาะสมของ visual style กับกลุ่มเป้าหมาย
- ความสมเหตุสมผลของ **user flow ในเชิงความหมาย** (กราฟถูกต้อง ≠ flow สมเหตุสมผล)
- ความคิดริเริ่ม / ความแปลกใหม่

> **นัยเชิงสถาปัตยกรรม (ส่งต่อ ticket 09 และ 17):** ควรแยกเป็น 2 ชั้นชัดเจน — **ชั้น deterministic analyzer** (คำนวณตาราง A–H ข้างบน ผลิต metrics JSON) และ **ชั้น LLM judge** ที่รับ *ทั้ง* ภาพ render + metrics JSON ไปตัดสิน ไม่ใช่ให้ LLM นับเองจากภาพ ซึ่งจะทั้งช้า แพง และไม่นิ่ง

---

## 15. ทางถอย ถ้าเข้าถึง REST API ไม่ได้

| ทางเลือก | ได้อะไร | เสียอะไร |
|---|---|---|
| **นศ. export PNG/PDF แนบมา** | ได้ภาพให้ LLM ดู; ไม่ต้องมีสิทธิ์อะไรเลย | **เสียข้อมูลโครงสร้างทั้งหมด** — ตาราง A–H ข้างบนทำไม่ได้เลยยกเว้นสิ่งที่วัดจากภาพ (CV) ซึ่งแม่นน้อยกว่ามาก |
| **Figma plugin ฝั่ง นศ.** | Plugin API เข้าถึงเนื้อไฟล์ได้เต็ม แม้ไม่มีสิทธิ์ REST; export JSON แล้วให้ นศ. อัปโหลดเข้าระบบ | นศ. ต้องติดตั้ง plugin และรันเอง; **ต้องเปิด Figma อยู่** (plugin ทำงานอัตโนมัติไม่ได้); ต้องพัฒนา+เผยแพร่ plugin เพิ่ม; **นศ. แก้ไข JSON ก่อนส่งได้** = ความน่าเชื่อถือลดลง |
| **ให้ นศ. duplicate ไฟล์เข้า team ของวิชา** | ได้สิทธิ์เต็มผ่าน REST | นศ. ต้องทำเอง; ไฟล์แยกจากต้นฉบับ ทำให้ track ความคืบหน้ายาก |
| **`.fig` file** | — | **ไม่มี public spec** สำหรับ parse `.fig`; Figma ไม่ได้เผยแพร่รูปแบบไฟล์ — **ไม่แนะนำ** |

อ้างอิงการเปรียบเทียบ API: https://developers.figma.com/compare-apis/
— REST API "largely read-only, except for comments, comment reactions, variables, and dev resources"; Plugin API แก้ไขไฟล์ได้เต็มแต่ "require Figma to be open"

**ข้อเสนอ:** ให้ REST API เป็นทางหลัก และ **บังคับให้ นศ. แนบ PDF/PNG export มาด้วยเสมอ** เป็น fallback + เป็นหลักฐานสำเร็จรูปเวลาอาจารย์ทวนผล (สอดคล้องกับ map ที่ระบุว่าชิ้นงาน = Figma link + ไฟล์เอกสาร อยู่แล้ว)

---

## 16. สรุปสิ่งที่ยัง UNVERIFIED (ต้อง spike ก่อน commit สถาปัตยกรรม)

1. **ไฟล์ public view link อ่านผ่าน REST ด้วย PAT ของคนนอกได้หรือไม่** — สำคัญที่สุด เอกสารทางการไม่ระบุ
2. โครงสร้างภายในของ `interactions` (Reaction/Trigger/Action) ที่ REST serialize ออกมาจริง
3. ชื่อ key ย่อยของ `styles` ระดับ node (`fill`, `text`, `effect`, `grid`, `stroke`)
4. `boundVariables` โผล่ใน file response ของบัญชี non-Enterprise หรือไม่
5. ตัวเลข rate limit คอลัมน์ Enterprise (Starter/Professional ยืนยันแล้ว)
6. education plan ถูกจำกัด API ต่างจาก Professional หรือไม่ (help page ไม่พูดถึง)

**Spike ที่แนะนำ:** สร้างไฟล์ทดสอบ 1 ไฟล์ที่มี auto-layout + text styles + component + prototype แล้ว dump `GET /v1/files/:key` เต็ม ๆ ลงไฟล์ อ่านด้วยตา 30 นาที — จะเคลียร์ข้อ 2, 3, 4 พร้อมกัน และได้ fixture ไว้เขียน test ของ analyzer ด้วย

---

## แหล่งอ้างอิงทั้งหมด

**Figma official developer docs**
- https://developers.figma.com/docs/rest-api/ — Introduction
- https://developers.figma.com/docs/rest-api/file-endpoints/ — GET file / nodes / images / image fills / meta
- https://developers.figma.com/docs/rest-api/files/ — Global node properties
- https://developers.figma.com/docs/rest-api/file-node-types/ — Node types + prototype properties
- https://developers.figma.com/docs/rest-api/component-types/ — Component / ComponentSet / Style
- https://developers.figma.com/docs/rest-api/comments-endpoints/ — Comments endpoints
- https://developers.figma.com/docs/rest-api/comments-types/ — Comment object
- https://developers.figma.com/docs/rest-api/version-history-endpoints/ — Version history
- https://developers.figma.com/docs/rest-api/webhooks/ — Webhooks V2
- https://developers.figma.com/docs/rest-api/authentication/ — Authentication overview
- https://developers.figma.com/docs/rest-api/personal-access-tokens/ — PAT
- https://developers.figma.com/docs/rest-api/oauth-apps/ — OAuth 2
- https://developers.figma.com/docs/rest-api/plan-access-tokens/ — Plan access tokens
- https://developers.figma.com/docs/rest-api/scopes/ — Scopes
- https://developers.figma.com/docs/rest-api/rate-limits/ — Rate limits
- https://developers.figma.com/docs/rest-api/errors/ — Error codes
- https://developers.figma.com/compare-apis/ — REST vs Plugin vs Widget API
- https://developers.figma.com/docs/plugins/api/Reaction/ — Reaction (Plugin API)

**Figma official OpenAPI spec (repo ของ Figma เอง)**
- https://raw.githubusercontent.com/figma/rest-api-spec/main/dist/api_types.ts — TypeStyle, RGBA, SolidPaint, HasLayoutTrait, Component, Style, FrameOffset

**Figma Help Center**
- https://help.figma.com/hc/en-us/articles/8085703771159-Manage-personal-access-tokens
- https://help.figma.com/hc/en-us/articles/360039960434-Manage-seats-in-Figma
- https://help.figma.com/hc/en-us/articles/360041061214-Figma-for-Education
- https://help.figma.com/hc/en-us/articles/1500007609322-Guide-to-sharing-and-permissions
- https://www.figma.com/pricing/
- https://www.figma.com/education/

**Community (ใช้ประกอบ ไม่ใช่หลักฐานชี้ขาด)**
- https://forum.figma.com/ask-the-community-7/rest-api-how-can-i-access-a-password-protected-file-using-a-figma-api-by-passing-password-49020
