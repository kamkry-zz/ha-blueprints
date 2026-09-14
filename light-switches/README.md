# HA Blueprints

Home Assistant blueprints used in the `krysdom` house setup.

## `light-switches/mirror-devices-via-virtual-switch.yaml`

Mirrors two `switch`/`light` entities through a virtual `input_boolean` that acts
as the single source of truth. Unlike a plain state mirror, it is designed for
flaky local Tuya devices:

- **Dashboard control** – toggling the virtual helper drives both devices.
- **Physical control** – changing either device (wall gang or smart plug) updates
  the virtual helper, so the last real change always wins.
- **Reconnect handling** – reports that arrive after a device was
  `unavailable`/`unknown` are accepted when the virtual helper was last changed
  *before* the device went offline (a physical press during the outage). They are
  ignored when the virtual was changed *while* the device was offline (the
  command may have been lost), so the original "lamp comes back on 20-30 s later"
  flip cannot happen.
- **Debounce** – device changes must stay stable for `settle_seconds`
  (default 0.25 s) before they are accepted.

There is intentionally **no periodic reconciliation**: with flaky devices it
races fresh state reports and its commands can be delivered late (turning relays
on minutes later). A lost command is resolved by the next user action or device
report.

One helper and one automation per pair.

### Inputs

| Input | Description | Default |
|---|---|---|
| `virtual_switch` | `input_boolean` acting as the source of truth | – |
| `device_a` / `device_b` | The two physical `switch`/`light` entities | – |
| `settle_seconds` | Debounce before a device change is accepted | 0.25 |

### Pairs in this house

| Virtual helper | Device A | Device B |
|---|---|---|
| `input_boolean.kamil_lampa` | `switch.kamil_lampa_podlogowa` | `switch.switch_right` |
| `input_boolean.salon_przy_tv` | `switch.salon_przy_tv` | `switch.switch_center` |
| `input_boolean.salon_przy_lozku` | `switch.salon_przy_lozku` | `switch.switch_right_2` |
| `input_boolean.korytarz_drzwi` | `switch.switch_left_3` | `switch.switch_right_4` |
| `input_boolean.korytarz_srodek` | `switch.switch_left_4` | `switch.switch_right_3` |
| `input_boolean.sypialnia_lampka` | `switch.sypialnia_lampka` | `switch.switch_right_5` |
| `input_boolean.sypialnia_biurko` | `switch.sypialnia_biurko` | `switch.switch_center_2` |
| `input_boolean.sypialnia_biblioteczka` | `switch.sypialnia_biblioteczka` | `switch.switch_right_5` |

`switch.switch_right_5` intentionally appears in two pairs: the physical gang
controls both Sypialnia lamps.

The helpers are declared in the `krysdom-api` chart
(`home-assistant/templates/configuration-configmap.yaml`, `helpers.yaml`) and are
restored across Home Assistant restarts.

### Deploying

1. Copy the blueprint to the Home Assistant config volume:
   `/config/blueprints/automation/kamkry-zz/mirror-devices-via-virtual-switch.yaml`.
2. Restart Home Assistant (or reload blueprints) and create one automation per
   pair from the blueprint, pointing it at the pair's helper and two devices.
3. Point the dashboard card for each pair at the helper entity.
