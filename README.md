# MakerWorld Creator Rewards & Stats for Home Assistant

A guide to tracking **MakerWorld points**, **gift card redemption progress** & **earning velocity** in Home Assistant using your existing Bambu Lab ([`ha-bambulab`](https://github.com/greghesp/ha-bambulab)) integration.

---

## How Bambu Lab Entity IDs are Formatted

When Home Assistant creates entities through ([`ha-bambulab`](https://github.com/greghesp/ha-bambulab)), it formats sensor entity IDs using your assigned **Area/Location** (if set) and your **MakerWorld Username/Handle**:

```text
sensor.{location}_makerworld_{user_id}_{metric}
```

* `{location}`: The Home Assistant Area/Room your printer or profile is assigned to (e.g. `workshop`, `3d_printing`, or omitted if no area is assigned).
* `{user_id}`: Your MakerWorld username or handle (e.g. `maker123`, `creator_name`).
* `{metric}`: The statistic (e.g. `points`, `boost_tokens`, `downloads_incl_print_profiles`).

> **How to find your exact entity names:**  
> Go to **Home Assistant** → **Developer Tools** → **States** tab and type `makerworld` or `points` into the filter box. You will see all your sensors listed there.

---

## Sensor Directory & Architecture

```
┌────────────────────────────────────────────────────────┐
│  Bambu Lab / MakerWorld Cloud                         │
│  (Existing ha-bambulab Integration)                   │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
 ┌───────────────────────────────────────────────────────┐
 │ 1. EXISTING INTEGRATION SENSORS                       │
 │  • sensor.{location}_makerworld_{user_id}_points      │
 │  • sensor.{location}_makerworld_{user_id}_boost_...   │
 │  • sensor.{location}_makerworld_{user_id}_downl...    │
 └─────────────┬───────────────────────────┬─────────────┘
               │                           │
               ▼                           ▼
 ┌───────────────────────────┐ ┌─────────────────────────┐
 │ 2. UI INPUT HELPERS       │ │ 3. UI DERIVATIVE HELPER │
 │  • Points Required (535)  │ │  • 7-Day Velocity       │
 │  • Voucher Value (35)     │ │    (pts / day)          │
 │  • Currency Dropdown (£)  │ └───────────┬─────────────┘
 └─────────────┬─────────────┘             │
               │                           │
               └─────────────┬─────────────┘
                             │
                             ▼
 ┌───────────────────────────────────────────────────────┐
 │ 4. NEW TEMPLATE SENSORS (template.yaml)               │
 │  • sensor.makerworld_available_gift_cards             │
 │  • sensor.makerworld_redeemable_cash_value            │
 │  • sensor.makerworld_points_to_next_gift_card         │
 │  • sensor.makerworld_gift_card_progress               │
 │  • sensor.makerworld_days_to_next_gift_card           │
 └───────────────────────────────────────────────────────┘
```

---

## Entity Breakdown

### 1. Existing Sensors from `ha-bambulab` (Find these in Developer Tools)
| Entity ID Placeholder | Metric Tracked |
| :--- | :--- |
| `sensor.{location}_makerworld_{user_id}_points` | Live MakerWorld points balance |
| `sensor.{location}_makerworld_{user_id}_boost_tokens` | Boost tokens in your wallet available to give |
| `sensor.{location}_makerworld_{user_id}_boosts_received` | Total boosts awarded to your models by users |
| `sensor.{location}_makerworld_{user_id}_downloads_incl_print_profiles` | Total downloads across models and print profiles |
| `sensor.{location}_makerworld_{user_id}_prints_incl_print_profiles` | Total prints across models and print profiles |
| `sensor.{location}_makerworld_{user_id}_likes` | Total model likes |
| `sensor.{location}_makerworld_{user_id}_collects` | Total model collections |
| `sensor.{location}_makerworld_{user_id}_followers` | Total profile followers |
| `sensor.{location}_makerworld_{user_id}_level` | MakerWorld creator level |

### 2. GUI Helpers Created in Home Assistant
| Entity ID | Type | Purpose |
| :--- | :--- | :--- |
| `input_number.makerworld_gift_card_points` | Number Helper | Points required per voucher (e.g. `535` for UK, `490` for US) |
| `input_number.makerworld_gift_card_value` | Number Helper | Numerical value of the voucher (e.g. `35` or `40`) |
| `input_select.makerworld_currency` | Dropdown Helper | Regional currency selector (All 10 MakerWorld regions) |
| `sensor.{location}_makerworld_{user_id}_makerworld_points_velocity` | Derivative Helper | Rolling points earned per day (`pts/day`) over a 7-day window |

### 3. New Calculated Template Sensors (Created in `template.yaml`)
| Entity ID | Unit | Purpose |
| :--- | :--- | :--- |
| `sensor.makerworld_available_gift_cards` | `cards` | Total vouchers ready to redeem right now (`pts // target`) |
| `sensor.makerworld_redeemable_cash_value` | *(Dynamic)* | Total wallet money ready to spend with symbol (e.g. `£0` or `$40`) |
| `sensor.makerworld_points_to_next_gift_card` | `pts` | Points remaining to reach the next card (`target - remainder`) |
| `sensor.makerworld_gift_card_progress` | `%` | Progress toward the next voucher milestone (`0.0% - 100.0%`) |
| `sensor.makerworld_days_to_next_gift_card` | `days` | Estimated days until next card (`points_needed / velocity`) |

---

### Supported MakerWorld Store Regions & Currencies

MakerWorld supports 10 distinct regional store redemption options:

| Code | Region / Store | Currency Symbol | Dropdown Option to Add |
| :--- | :--- | :--- | :--- |
| 🇬🇧 **UK** | United Kingdom | `£` (GBP) | `£ - UK` |
| 🇺🇸 **USA** | United States | `$` (USD) | `$ - USA` |
| 🇪🇺 **EU** | European Union | `€` (EUR) | `€ - EU` |
| 🇨🇦 **CA** | Canada | `CA$` (CAD) | `CA$ - CA` |
| 🇦🇺 **AU** | Australia | `AU$` (AUD) | `AU$ - AU` |
| 🇰🇷 **KR** | South Korea | `₩` (KRW) | `₩ - KR` |
| 🇯🇵 **JP** | Japan | `¥` (JPY) | `¥ - JP` |
| 🇲🇽 **MX** | Mexico | `MX$` (MXN) | `MX$ - MX` |
| 🇨🇳 **CN** | China Mainland | `¥` (CNY) | `¥ - CN` |
| **ASIA** | Singapore, Thailand, Malaysia, HK, MO, TW | `$` (USD) | `$ - ASIA` |

> ⚠️ **Asia Region Note:** Gift cards redeemed under Asia are only valid for use in Singapore, Thailand, Malaysia, and China (Hong Kong, Macao, Taiwan), and cannot be changed after redemption. Review will be completed within 5 business days.

---

## 🛠️ Step-by-Step Setup Guide

### Step 1: Create the UI Helpers

In Home Assistant, navigate to **Settings** → **Devices & Services** → **Helpers** tab.

1. **Points Required Helper (`input_number`)**:
   * Click **+ Create Helper** → **Number**
   * **Name:** `MakerWorld Gift Card Points`
   * **Icon:** `mdi:target`
   * **Min:** `50` | **Max:** `10000` | **Step:** `1`
   * **Unit of measurement:** `pts`
   * **Display Mode:** `Input box` (recommended over Slider for precise entry)
   * *Initial Value:* `535` (UK £35 threshold) or `490` (US $40 threshold)

2. **Voucher Value Helper (`input_number`)**:
   * Click **+ Create Helper** → **Number**
   * **Name:** `MakerWorld Gift Card Value`
   * **Icon:** `mdi:cash`
   * **Min:** `1` | **Max:** `100000` | **Step:** `1` *(allows large values for currencies like KRW ₩ or JPY ¥)*
   * **Display Mode:** `Input box`
   * *Initial Value:* `35` (or `40`)

3. **Currency Dropdown Helper (`input_select`)**:
   * Click **+ Create Helper** → **Dropdown**
   * **Name:** `MakerWorld Currency`
   * **Icon:** `mdi:currency-usd`
   * **Options:** Add the 10 official MakerWorld regional options:
     ```text
     £ - UK
     $ - USA
     € - EU
     CA$ - CA
     AU$ - AU
     ₩ - KR
     ¥ - JP
     MX$ - MX
     ¥ - CN
     $ - ASIA
     ```
   * *Initial Selection:* Choose your region (e.g. `£ - UK`)
   * *(Alternatively, you can use a **Text** helper `input_text.makerworld_currency` if you prefer typing freeform text).*

4. **Earning Velocity Helper (`derivative`)**:
   * Click **+ Create Helper** → **Derivative**
   * **Input sensor:** Select your MakerWorld Points sensor (`sensor.{location}_makerworld_{user_id}_points`)
   * **Time window:** `168:00:00` (168 hours = 7 days)
   * **Time unit:** `Days`
   * **Precision:** `2`
   * *Note the resulting entity ID:* It will typically be named `sensor.{location}_makerworld_{user_id}_makerworld_points_velocity` or `sensor.{location}_makerworld_{user_id}_points_derivative`.
   * *Sensor will read "0" until points are nexts added to the account*

---

### Step 2: Add New Template Sensors to `template.yaml`

Open your **`template.yaml`** file and paste the following block.

> ⚠️ **Important:** Replace `sensor.{location}_makerworld_{user_id}_points` and `sensor.{location}_makerworld_{user_id}_makerworld_points_velocity` with your actual sensor names from Step 1.

```yaml
- sensor:
    # 1. Total available vouchers ready to redeem right now
    - name: "MakerWorld Available Gift Cards"
      unique_id: makerworld_available_gift_cards
      icon: mdi:gift
      unit_of_measurement: "cards"
      state: >
        {% set pts = states('sensor.{location}_makerworld_{user_id}_points') | int(0) %}
        {% set target = states('input_number.makerworld_gift_card_points') | int(535) %}
        {{ pts // target if target > 0 else 0 }}

    # 2. Total currency value ready to spend (e.g. £0, £35, $40, €40)
    - name: "MakerWorld Redeemable Cash Value"
      unique_id: makerworld_redeemable_cash_value
      icon: mdi:cash-multiple
      state: >
        {% set cards = states('sensor.makerworld_available_gift_cards') | int(0) %}
        {% set card_val = states('input_number.makerworld_gift_card_value') | int(35) %}
        {% set raw_curr = states('input_select.makerworld_currency') | default('£ - UK') %}
        {% set symbol = raw_curr.split(' ')[0] if ' ' in raw_curr else raw_curr %}
        {{ symbol }}{{ cards * card_val }}
      attributes:
        raw_amount: >
          {% set cards = states('sensor.makerworld_available_gift_cards') | int(0) %}
          {% set card_val = states('input_number.makerworld_gift_card_value') | int(35) %}
          {{ cards * card_val }}
        currency_symbol: >
          {% set raw_curr = states('input_select.makerworld_currency') | default('£ - UK') %}
          {{ raw_curr.split(' ')[0] if ' ' in raw_curr else raw_curr }}
        selected_region: "{{ states('input_select.makerworld_currency') | default('£ - UK') }}"

    # 3. Points needed until your next voucher (e.g. 535 - 332 = 203 pts)
    - name: "MakerWorld Points to Next Gift Card"
      unique_id: makerworld_points_to_next_card
      icon: mdi:target
      unit_of_measurement: "pts"
      state: >
        {% set pts = states('sensor.{location}_makerworld_{user_id}_points') | int(0) %}
        {% set target = states('input_number.makerworld_gift_card_points') | int(535) %}
        {{ target - (pts % target) if target > 0 else 0 }}

    # 4. Progress percentage for gauge display (332 / 535 = 62.1%)
    - name: "MakerWorld Gift Card Progress"
      unique_id: makerworld_gift_card_progress
      icon: mdi:percent
      unit_of_measurement: "%"
      state: >
        {% set pts = states('sensor.{location}_makerworld_{user_id}_points') | int(0) %}
        {% set target = states('input_number.makerworld_gift_card_points') | int(535) %}
        {{ (((pts % target) / target) * 100) | round(1) if target > 0 else 100 }}

    # 5. Estimated days remaining until next voucher based on 7-day velocity
    - name: "MakerWorld Days to Next Gift Card"
      unique_id: makerworld_days_to_next_card
      icon: mdi:calendar-clock
      state: >
        {% set pts_left = states('sensor.makerworld_points_to_next_gift_card') | float(0) %}
        {% set velocity = states('sensor.{location}_makerworld_{user_id}_makerworld_points_velocity') | float(0) %}
        {% if velocity > 0 %}
          {{ (pts_left / velocity) | round(1) }} days
        {% else %}
          Waiting for points...
        {% endif %}
```

> **Apply changes:** Go to **Developer Tools** → **YAML** → click **Template Entities** to reload without restarting Home Assistant.

---

### Step 3: Lovelace Dashboard Card YAML

Add a new Manual card to your dashboard and paste:

```yaml
type: vertical-stack
cards:
  - type: custom:mushroom-title-card
    title: MakerWorld Rewards
    subtitle: Bambu Lab Creator Tracker

  # Status Row: Gauge Progress + Ready Cash + Points Needed
  - type: horizontal-stack
    cards:
      - type: gauge
        entity: sensor.makerworld_gift_card_progress
        name: Next Card Progress
        min: 0
        max: 100
        needle: true
        severity:
          red: 0
          yellow: 50
          green: 80

      - type: vertical-stack
        cards:
          - type: custom:mushroom-entity-card
            entity: sensor.makerworld_redeemable_cash_value
            name: Ready to Spend
            icon: mdi:cash-check
            icon_color: green
          - type: custom:mushroom-entity-card
            entity: sensor.makerworld_points_to_next_gift_card
            name: Points Needed
            icon: mdi:target
            icon_color: amber

  # Creator Analytics List
  - type: entities
    title: Creator Analytics
    entities:
      - entity: sensor.{location}_makerworld_{user_id}_points
        name: Current Points
      - entity: sensor.{location}_makerworld_{user_id}_makerworld_points_velocity
        name: 7-Day Velocity
      - entity: sensor.makerworld_days_to_next_gift_card
        name: Est. Days Remaining

  # Interactive GUI Settings
  - type: entities
    title: ⚙️ Reward Settings
    entities:
      - entity: input_select.makerworld_currency
        name: Currency
      - entity: input_number.makerworld_gift_card_value
        name: Voucher Value
      - entity: input_number.makerworld_gift_card_points
        name: Points Required per Voucher
```
<img width="500" height="689" alt="image" src="https://github.com/user-attachments/assets/de78f43d-427d-47ac-a498-9fd643557104" />
---

### Step 4: Milestone Push Notification Automation

Trigger an alert to your phone as soon as your points cross the voucher threshold:

```yaml
alias: 'MakerWorld: Gift Card Ready'
triggers:
  - entity_id: sensor.makerworld_points_to_next_gift_card
    below: 1
    trigger: numeric_state
actions:
  - action: notify.send_message
    target:
      entity_id: notify.{entity}
    data:
      message: >-
        You have earned enough points for another Bambu Lab voucher! Total ready
        to spend: {{ states('sensor.makerworld_redeemable_cash_value') }}
```
