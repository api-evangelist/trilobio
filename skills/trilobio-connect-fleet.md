---
name: trilobio-connect-fleet
description: Connect to a Trilobio fleet from Python, check status, and place labware
  on a robot deck using the tcode-api Servicer client.
api: tcode-api
method: generated
source: http://tcode.trilo.bio/ (Tutorial: Connecting to a Trilobio Fleet; Entities)
operations:
- TCodeServicerClient()
- TCodeServicerClient.get_status
- TCodeServicerClient.serial_number_lookup
- CREATE_LABWARE
---

# Connect to a Trilobio Fleet

Drive a Trilobot fleet from Python with the `tcode-api` package. This skill must
run **on the fleet control computer** (the machine where the T-code Servicer is
running) — the client connects to the local Servicer and requires physical/network
access to that machine rather than API keys.

## Prerequisites
- Python 3.11+ with `tcode-api` installed (fleet controllers ship it pre-installed;
  otherwise `uv add "git+https://github.com/trilobio/tcode-api"`).
- A booted Trilobot fleet.

## Steps

1. **Connect to the Servicer.** Construct the client with no arguments; it connects
   to the T-code Servicer on the local machine.
   ```python
   from tcode_api.servicer import TCodeServicerClient
   client = TCodeServicerClient()
   ```

2. **Verify the fleet is reachable.** Call `get_status()` and confirm
   `result.success` is `True` before sending commands.
   ```python
   status = client.get_status()
   assert status.result.success, status.result.message
   ```

3. **Target a specific robot (optional).** In a multi-robot fleet, resolve a robot
   by serial number so subsequent commands address the intended machine.
   ```python
   client.serial_number_lookup(...)
   ```

4. **Place labware on the deck.** Use a `CREATE_LABWARE` command to add labware to a
   robot's deck (this is the only entity `CREATE_*` currently implemented; tools and
   robots are created at fleet-controller startup). Passing a `PipetteTipRack`
   description with `full=True` implicitly creates the pipette tips.

5. **Handle results.** Every response wraps a `Result(success, code, message, details)`.
   Check `success` and surface `code`/`message` on failure rather than assuming the
   command applied.

## Conventions
- No credentials — trust boundary is access to the fleet control computer.
- Message schemas are versioned and immutable per release; see
  `skills/trilobio-AGENTS.md` for the schema-migration policy.
- See `conventions/trilobio-conventions.yml` and `data-model/trilobio-data-model.yml`
  for the full command/entity model.
