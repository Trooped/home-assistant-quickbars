# QuickBars — Home Assistant Integration (HACS)
# NOTE: This integration is NOT active at the moment, the code is public anyway, but the app does NOT support the features here.

> Display overlays, PiP cameras, and rich notifications on **Android/Google TV** via the QuickBars TV app — controlled by your Home Assistant automations.

[![HACS Custom](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://hacs.xyz/)
[![Minimum HA](https://img.shields.io/badge/Home%20Assistant-2025.10%2B-41BDF5)](#requirements)
[![Add to HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=Trooped&repository=home-assistant-quickbars&category=integration)

---

## Why QuickBars?

- **Instant control**: open a QuickBar overlay on the TV with your favorite entities (lights, climate, scenes, etc.).
- **Camera PiP**: show/hide a camera as picture-in-picture anywhere on screen; supports entity MJPEG as well as **direct RTSP** URLs.
- **Actionable notifications**: send rich TV notifications with buttons and handle the resulting **events** in automations.
- **Local first**: runs entirely on your LAN with real-time push.
- **No entities created**: the integration exposes **services** and emits events; it doesn’t create entities in HA.

---

## Requirements

- **Home Assistant** 2025.10.0 or newer
- **Android/Google TV** (Android 9+) device with the **QuickBars for Home Assistant** TV app installed
- TV and Home Assistant on the **same LAN**
- Keep the TV app open in foreground during first pairing

---

## Install (HACS)

**Option A — Today (custom repo):**

1. In Home Assistant: **HACS → ⋮ (menu) → Custom repositories**.
2. Add `https://github.com/Trooped/home-assistant-quickbars` as **Integration**.
3. Search **QuickBars** in HACS → **Install** → **Restart Home Assistant**.

Or click:  
[![Add repo in HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=Trooped&repository=home-assistant-quickbars&category=integration)

**Option B — Later (default store):** once accepted into the HACS default store, just search “QuickBars” inside HACS.

---

## Setup

1. **Settings → Devices & Services → Add Integration → QuickBars**
2. If discovered via **Zeroconf**, just confirm the device; otherwise enter **TV IP** and **port** (default `9123`).
3. Enter the **pairing code** shown on the TV.
4. (Optional) Provide your HA URL + long-lived token to let the TV app talk back to HA for real-time actions.

> Tip: If discovery doesn’t show up, ensure mDNS/zeroconf is allowed on your network and the TV app is open.

---

## Services

### `quickbars.quickbar_toggle`
Open or close a **QuickBar overlay** by its alias (as configured in the TV app).

```yaml
service: quickbars.quickbar_toggle
data:
  alias: living_room            # required
target:
  device_id: 1234567890abcdef   # optional (broadcasts if omitted)
```

### `quickbars.camera_toggle`
Show/hide a **camera** as PiP. Use a Home Assistant camera **entity** (with MJPEG) or a direct **RTSP** URL.

```yaml
# Using a camera entity (MJPEG)
service: quickbars.camera_toggle
data:
  camera_entity: camera.front_door
  position: top_right           # top_left | top_right | bottom_left | bottom_right
  size: large                   # small | medium | large
  auto_hide: 25                 # seconds (0 = never)
target:
  device_id: abcdef123456
```

```yaml
# Using a direct RTSP URL (no camera entity needed)
service: quickbars.camera_toggle
data:
  rtsp_url: rtsp://user:pass@192.168.1.200:554/stream1
  position: bottom_left
  size_px:
    w: 640
    h: 360
target:
  device_id: abcdef123456
```

Optional fields:
- `show_title: true|false`
- `size` **or** `size_px: { w: <int>, h: <int> }`

### `quickbars.notify`
Send a **rich TV notification** with optional icon, image, sound, and **buttons**.

```yaml
service: quickbars.notify
data:
  title: "New Visitor!"
  message: "Someone is at the door"
  mdi_icon: mdi:doorbell
  length: 15
  position: bottom_left
  image:
    path: images/doorbell.jpg        # or url / media_id
  sound:
    path: chimes/ding.mp3            # or url / media_id
  sound_volume_percent: 120
  actions:
    - id: open
      label: Open
    - id: ignore
      label: Ignore
target:
  device_id: abcdef123456
```

---

## Events

When a user presses a button in a TV notification, the app fires a **Home Assistant event**:

- **Event type**: `quickbars.action`
- **Payload** includes: `action_id` (the id you supplied in `actions`)

Example automation:

```yaml
alias: "Handle TV Notification Action - Unlock Door"
triggers:
  - trigger: event
    event_type: quickbars.action
    event_data:
      action_id: unlock_door
actions:
  - action: lock.unlock
    target:
      entity_id: lock.front_door
```

---

## Example: Doorbell Flow

```yaml
alias: "Doorbell Pressed - Show on TV"
triggers:
  - trigger: state
    entity_id: binary_sensor.doorbell
    to: "on"
actions:
  - action: quickbars.camera_toggle
    data:
      device_id: abcdef123456
      camera_entity: camera.front_door
      position: top_right
      auto_hide: 25
  - action: quickbars.notify
    data:
      device_id: abcdef123456
      title: "Doorbell"
      message: "Someone is at the door"
      mdi_icon: mdi:doorbell
      length: 25
      actions:
        - id: open
          label: "Open"
        - id: ignore
          label: "Ignore"
```

---

## Troubleshooting

- **TV not reachable / pairing fails**
  - Ensure the TV app is **open in foreground** and both devices are on the same LAN.
  - Verify your network allows **mDNS/zeroconf**.
  - If entering an HA URL, use a **LAN-reachable** hostname/IP (not `localhost`).

- **No PiP / notification appears**
  - In the TV app, enable **“Persistent background connection.”**
  - For camera entities, ensure the entity provides **MJPEG**; or use an **RTSP** URL.

---

## Privacy & Locality

QuickBars communicates on your **local network** and does not require any cloud service. Commands are pushed over a persistent local connection for real-time control.

---

## Roadmap

- HACS default-store listing
- More camera layouts & presets
- Additional actions & overlay types
- (Longer-term) retry for official core inclusion

---

## Contributing

Issues and PRs welcome!  
- **Issues**: <https://github.com/Trooped/home-assistant-quickbars/issues>
- **Discussions**: ideas/feedback and UX suggestions are appreciated.

---

## License

Choose a license you prefer (MIT is common for HA custom integrations). Add `LICENSE` to the repo.

---

## FAQ

**Does this create HA entities?**  
No — it exposes **services** and emits events so you can build automations without clutter.

**Can I use both MJPEG and RTSP?**  
Yes. If you send both a camera entity and an `rtsp_url`, the TV app will prefer **RTSP** for that request.

**Does it work on non-Android TVs?**  
No. Android/Google TV only (permissions like “Display over other apps” / Accessibility are Android-specific).

---

## Dev Notes

- Folder: `custom_components/quickbars/`
- `manifest.json` **must** include `"version"` for custom integrations.
- Add `hacs.json` at repo root with your minimum HA version and `render_readme: true`.
