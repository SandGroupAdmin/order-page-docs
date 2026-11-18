# Order Page – Front-end Documentation (v6)

November 18, 2026

## Repository contents

| Path | What it is |
| --- | --- |
| `README.md` | This documentation |
| `css/order-page.css` | The CSS block to add to the admin stylesheet |
| `prototype/order-page-v6.html` | Standalone v6 prototype; open it in a browser |

The prototype includes the live admin's shell styles so it renders on its own. In the real admin, only `css/order-page.css` needs adding.

## Overview

This spec describes **Order page v6**: the redesigned single-order screen in the Gravelshop admin. It reuses the live admin shell (header, two-level sidebar, breadcrumb) and the existing stylesheet from `controller.asp`, so most classes already exist; the few new ones are listed under CSS reference.

The page answers one question per area, top to bottom:

| Area | Answers | Position |
| --- | --- | --- |
| Page header | Which order, what status | Top of content |
| Order head | Who ordered, where it goes, what they asked for | Main column, left |
| Payment & costs | What was paid, what it cost, what we earn | Main column, right |
| Send notifications | Which customer emails went out | Under Payment & costs |
| Items | What was ordered and what is confirmed | Main column, full width |
| Order item 1 | How it is sourced, hauled and priced | Main column, full width |
| Internal notes | What the team said about it | Right sidebar, full height |

Customer personal data is shown as placeholders such as `[Customer name]`; the backend fills the real values.

## Page structure

The page is the standard admin shell with a two-column content grid: order information on the left (`.stack-16`), Internal notes on the right (340px). Inside the left column, Order head and the Payment/Notifications stack sit side by side, then Items and Order item 1 run full width.

```html
<div class="admin">
  <header class="admin-header">              <!-- banner, logo, search, bell, user, quick links -->
    <div class="admin-header-art"></div>
    <div class="admin-header-row">…</div>
    <nav class="admin-header-links">…</nav>
  </header>

  <div class="admin-shell">
    <nav class="admin-sidebar">             <!-- 72px rail + Handling menu -->
      <div class="admin-rail">…</div>
      <div class="admin-panel">…</div>
    </nav>

    <main class="admin-content">
      <div class="admin-content-inner">
        <nav class="admin-breadcrumb">…</nav>
        <div class="admin-page-header">       <!-- title, subtitle, status dropdown, process badge -->
        <div class="viewas-bar">             <!-- Manual status -->

        <div class="op-layout">              <!-- main | notes -->
          <div class="stack-16">
            <div class="admin-grid admin-grid-2">
              <section class="admin-card">  <!-- Order head -->
              <div class="stack-16">
                <section class="admin-card"><!-- Payment & costs -->
                <section class="admin-card"><!-- Send notifications -->
              </div>
            </div>
            <section class="admin-card">    <!-- Items -->
            <section class="admin-card">    <!-- Order item 1 -->
          </div>
          <aside class="admin-card">        <!-- Internal notes -->
        </div>
      </div>
    </main>
  </div>
</div>
```

Every card uses the same three parts: `.admin-card` > `.admin-card-header` (title left, note or link right) > `.admin-card-body`.

## Design tokens

All values come from the live admin stylesheet; nothing new is introduced. Use the variables, never raw hex, in new rules.

```css
:root {
  --green: #0f7b46;      --green-dark: #086638;
  --blue: #2784d9;       --yellow: #e6a51a;      --red: #dc4c4c;
  --text: #14213a;       --text-light: #65738a;
  --border: #dfe5ec;     --bg: #f5f7fa;
  --radius: 12px;        --radius-sm: 8px;
  --shadow: 0 4px 18px rgba(25,45,70,.06);
  --sidebar: 272px;
}
```

| Token | Value | Used for |
| --- | --- | --- |
| Font | -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial | All text |
| Base size | 14px | Body text |
| Page title | 33px, letter-spacing -0.8px | `.admin-page-title` |
| Card title | 17px, 700 | `.admin-card-title` |
| Data text | 13px | Tables, key/value rows, notes |
| Labels | 11–12px, 600–700 | Section labels, table headers |
| Logo | Anton 18px, letter-spacing 5px, scaleY(1.5) | `.admin-logo-anton` |
| Card | radius 12px, 1px `--border`, `--shadow` | `.admin-card` |
| Inner panel | radius 8px, 1px `--border` | `.op-panel`, `.oi-col`, `.oi-strip` |
| Divider | 1px #edf0f3 | Rows inside cards |
| Grid gap | 16px | Between cards |
| Breakpoint | 1200px | Two columns collapse to one |

Status colors reuse the existing badges: green `#dcf5e4 / #147b3b` (done, paid, sent), red `#fde2e2 / #c93838` (not sent), yellow `#fff0c9 / #9c6a09` (waiting), grey `#edf0f3 / #626d79` (neutral).

## Components

Header, sidebar and breadcrumb are the shared admin includes, unchanged. Only the page-specific parts are listed here.

### Page header and status

Title and subtitle on the left; the status dropdown and process badge on the right. Manual status sits below as a compact yellow bar.

```html
<div class="admin-page-header">
  <div>
    <h1 class="admin-page-title">Order #11303842</h1>
    <div class="admin-page-subtitle">Created … · 14 ton … · Delivery …</div>
  </div>
  <div class="ws-head-actions">
    <label for="order-status" class="sr-only">Order status</label>
    <select id="order-status" class="status-select">…</select>
    <span class="badge badge-green">Process 250 Completed</span>
  </div>
</div>
<div class="viewas-bar" role="status">
  <svg>…</svg><b>Manual status</b><span>(flo) Set up. (10/11)</span>
  <button type="button">Clear</button>
</div>
```

### Order head

Key/value rows for contact data, then Billing and Delivery side by side, then the customer note.

```html
<section class="admin-card">
  <div class="admin-card-header"><div class="admin-card-title">Order head</div><span class="admin-card-note">#11303842</span></div>
  <div class="admin-card-body">
    <div class="kv-list"><div><b>Email</b><span>…</span></div> …</div>
    <div class="oh-addr">
      <div><div class="admin-menu-sub">Billing</div><span>…</span></div>
      <div><div class="admin-menu-sub">Delivery</div><span>…</span><a class="oh-map">Open in map</a></div>
    </div>
    <div class="ct-suggest-banner"><strong>Customer note</strong>…</div>
  </div>
</section>
```

### Payment & costs

Three blocks in one card: payment and profit (`.donut-table`), Customer price (`.donut-table` under an `.admin-menu-sub` label), then the Stripe transaction (`.kv-list`). Totals rows use `tr.is-total`. "Set unpaid" is the header link (`.admin-card-link`).

### Send notifications

One full-width `.admin-shortcut` button per message. A sent message shows the existing green chip.

```html
<button type="button" class="admin-shortcut">
  <span>Payment has been received</span>
  <span class="ac-tipchip is-sent"><svg>…</svg>Sent</span>
</button>
```

Only three buttons: Payment has been received, Order confirmation, Letter of liability link.

### Items

A standard `.admin-table` inside `.scroll-x`. Columns: Amount, Product, Method, Supplier · transport, Material, Transport, Delivery. Confirmation columns use `badge-red` "Not sent" or `badge-green` "Confirmed". The delivery day uses `.ws-tomorrow`.

### Order item 1

Four stacked parts inside one card:

1. **Delivery strip** (`.oi-strip`): one `.oi-strip-row` for the delivery time with the date select and Change button, one for the route (km, minutes, from → to).
2. **Material | Transport** (`.admin-grid-2` of two `.oi-col`): each has a grey header bar (`.oi-col-head`), a bold title (`.oi-title`), sub lines (`.oi-sub`) and a definition list (`.oi-dl`) with the company dropdowns and contact.
3. **Pricing** (`.admin-grid-2` of two `.op-panel.op-panel-soft`): Product pricing and Transport pricing & totals, each a `.donut-table`, with the Recalculate links (`.op-links`).
4. **Actions** (`.script-actions`): five equal `.quick-link` buttons, Make/Send one company pdf, Make/Send pickup pdf, Send haul order.

Contact markup (both columns):

```html
<dt>Contact</dt>
<dd class="oi-contact">
  <span class="oi-contact-name">[Contact name]</span>
  <a href="tel:8123338560"><svg>…</svg>812-333-8560</a>
  <a href="mailto:…"><svg>…</svg>[contact email]</a>
</dd>
```

### Internal notes

An `<aside class="admin-card">` in the right column: `.notepad-text` textarea, `.notepad-actions` (Pin to top checkbox, Save), then `ul.note-list`. Human notes are `li.note-item`; automatic status changes add `.note-auto`, which greys them out and hides the delete button.

## CSS reference

The rules to add live in [`css/order-page.css`](css/order-page.css). Add them as one new block after the existing sections of the admin stylesheet. Everything else on the page is already there.

**Reused unchanged:** `admin-*` shell and card classes, `kv-list`, `donut-table`, `num`, `pct`, `admin-table`, `badge-*`, `ws-tomorrow`, `ws-select`, `ws-head-actions`, `viewas-bar`, `ct-suggest-banner`, `admin-menu-sub`, `admin-shortcut`, `ac-tipchip`, `quick-link`, `script-actions`, `btn-save`, `form-input`, `notepad-text`, `notepad-actions`, `note-list`, `note-item`, `in-check`, `text-muted`, `text-red`, `text-green`.

The prototype file still contains a few rules from earlier design versions (`.id-chip`, `.oi-tiles`, `.oi-totals`, `.badge-orange`, `.oi-note`). They are unused in v6 and should not be copied.

## Data mapping

Every value on the page and how it is shown. Money is always two decimals with a period (641.27); dates stay in the admin's DD-MM-YYYY format.

| Area | Field | Example | Format rule |
| --- | --- | --- | --- |
| Page header | Order ID | 11303842 | Title as "Order #ID" |
| Page header | Status | Waiting delivery Thursday | Dropdown, current status selected |
| Page header | Process | 250 Completed | `badge-green` |
| Page header | Manual status | (flo) Set up. (10/11) | `viewas-bar`; hidden when empty |
| Order head | Created | 05-11-2026 21:34 | DD-MM-YYYY HH:MM |
| Order head | Company | — | Em dash when empty |
| Order head | Email, Phone | [Customer email] | Plain text; phone followed by area in brackets |
| Order head | IP | [IP address] (US-IN) | IP followed by country-state |
| Order head | Payment | Stripe | Method name |
| Order head | Billing, Delivery | 3-line address | Name, street, ZIP + city |
| Order head | Customer note | Drop-off instructions | `ct-suggest-banner`; hidden when empty |
| Payment & costs | Payment date | 05-11-2026 | DD-MM-YYYY |
| Payment & costs | Payment, Sales tax, Order costs, Profit | 641.27 / 44.89 / 570.00 / 71.27 | Money |
| Payment & costs | Profit % | 11.11% | Two decimals, period |
| Payment & costs | Registered paid | yes (method 10) | `badge-green` + method in `.pct` |
| Payment & costs | County, Sum plus tax, Sum tax free, Tax (7%), Price incl. tax | Brown County, IN / 641.27 / 0.00 / 44.89 / 686.16 | Money; tax rate in the label |
| Payment & costs | Transaction | 05-11-2026 21:34:26 · Stripe · 68616 | Date · provider · amount |
| Notifications | Sent state | Sent | `ac-tipchip is-sent` when sent |
| Items | Amount, Product, Method | 14 ton, Driveway Gravel #57 Gray (3/4"), Tipload (ton) | Plain text |
| Items | Material / Transport confirmed | Not sent | `badge-red` until confirmed |
| Items | Delivery | Thursday 12-11-2026 08:00 | Day name in `.ws-tomorrow` |
| Order item | Delivery window | Thursday morning · 8:00 AM – 12:00 PM · 11/12/2026 | As stored |
| Order item | Route | 44.9 km · 43 mins, from → to | km to one decimal; no meters |
| Order item | Material | product, amount, subname, INDname | Title + sub lines |
| Order item | Supplier, Pickup, Company, From | Rogers Group – Indiana | Dropdowns; no IDs shown |
| Order item | Account terms | Online account · net 30 | `.oi-hint` under the dropdown |
| Order item | Contact | name, phone, email | `tel:` and `mailto:` links |
| Order item | Product buy, sell, sum, profit | 18.90 / 25.33 / 264.60 / 90.02 | Money; profit in green |
| Order item | Transport cost, profit | 220.50 / 66.15 | Money; profit in green |
| Order item | Price total, Tax total | 641.27 / 44.89 | Money; tax total in red |
| Internal notes | Author, time, text | flo · 10-11-2026 14:12 | Newest first; system entries use `.note-auto` |

## Behavior and interactions

The prototype is static; this is what each control should do in the live page.

| Control | Location | Action |
| --- | --- | --- |
| Status dropdown | Page header | Saves the new order status on change; adds a "Status changed to" entry to Internal notes |
| Clear | Manual status bar | Clears the manual status and hides the bar |
| Set unpaid | Payment & costs header | Marks the order unpaid (same as the old "set unpaid" link) |
| Order costs | Payment & costs | Opens the order's cost lines in Controller |
| Send notification buttons | Send notifications | Sends that email; the button then shows the green Sent chip |
| Open in map | Order head, Delivery | Opens the delivery address in a map tab |
| Delivery time select + Change | Order item, delivery strip | Saves the new delivery slot |
| Supplier / Pickup / Company / From | Order item columns | Choose the company; saving recalculates prices |
| Phone, email | Order item contact | `tel:` and `mailto:` links |
| Alternatives, Orders in area | Material column | Same targets as today |
| Recalculate sum / tax | Transport pricing panel | Recalculates and refreshes the totals |
| Make / Send one company pdf, Make / Send pickup pdf, Send haul order | Order item actions | Same actions as today; all five are equal-weight buttons |
| Save, Pin to top | Internal notes | Adds the note; pinned notes stay at the top of the list |
| × on a note | Internal notes | Deletes a human note; system entries cannot be deleted |

Any action that sends something to a customer or supplier should show a short confirmation ("Haul order sent to Rogers Group") so staff know it went out.

## Responsive and accessibility

The page follows the admin's existing breakpoint: below 1200px every two-column grid becomes one column, and the page never scrolls sideways.

| Width | Layout |
| --- | --- |
| 1200px and up | Order info + Internal notes side by side; Order head next to Payment & costs; Material next to Transport; pricing panels side by side |
| Below 1200px | Everything stacks in one column: Order head, Payment & costs, Send notifications, Items, Order item 1, Internal notes |
| Any width | Items table scrolls inside `.scroll-x`; header quick links and strip rows wrap |

**Accessibility rules**

- Every select and textarea has a label; visually hidden ones use `.sr-only` (order status, delivery time, company dropdowns, new note).
- Icon-only buttons carry `aria-label` (notifications bell, account menu, delete note); decorative icons use `aria-hidden="true"`.
- Status is never shown by color alone: badges and chips always carry a word (Sent, Not sent, yes).
- Text on tinted backgrounds keeps at least 4.5:1 contrast; the badge color pairs above meet this.
- Keyboard focus uses the admin's green focus ring (`:focus-visible`); the status dropdown uses a yellow ring to match its fill.
- Cards are `<section>` with `aria-labelledby` pointing at their title; Internal notes is an `<aside>`; the manual status bar has `role="status"`.

## Implementation checklist

- [ ] Add `css/order-page.css` to the admin stylesheet
- [ ] Build the page inside the shared header, sidebar and breadcrumb includes
- [ ] Replace the old top status bar and yellow manual-status banner with the page header and `viewas-bar`
- [ ] Move Internal notes into the right-hand `<aside>`
- [ ] Merge the old separate Payment row into Payment & costs
- [ ] Show only three notification buttons
- [ ] Format all money with two decimals and a period
- [ ] Turn Supplier, Pickup, Company and From into dropdowns and hide internal IDs
- [ ] Show contact phone and email as links instead of "NEED TO RING IN"
- [ ] Make the five order item actions equal-weight buttons
- [ ] Use the time-of-day banner image already in the header include
- [ ] Check the page at 1440px, 1200px and below

**Open questions**

- The page file name is assumed to be `order.asp`; rename the CSS comment if it differs.
- The delivery window shows 11/12/2026 (US) while other dates use DD-MM-YYYY; pick one format for the whole page.
- Where does a contact's phone and email come from when a supplier has several contacts?
- Should "Need to ring the order in" still be flagged somewhere now that the badge is gone?
