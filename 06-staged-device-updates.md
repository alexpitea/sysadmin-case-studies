# Updating devices in the field without bricking them

## Context

A building-automation product has controllers installed on site, plus a desktop app and phone apps that talk to them. All of them need updates. A failed update of a controller is not an inconvenience: the device is in a ceiling or a technical room in another city, and it runs the heating.

The update system had to make a bad release cheap to survive.

## Principles

**One place publishes, and it keeps a record.** Releases are published to an update server and rolled out from there to chosen targets, one step at a time. Every rollout has an identifier, so it is always possible to say which device received what and when.

**Verify before trusting.** Every release has a manifest with the version, the size and a SHA-256 of the package. A package that does not match is discarded before anything is unpacked. The desktop app checks its own updates the same way and refuses a manifest that comes from a different origin or does not carry a newer version.

**One device first.** A release is rolled out in steps. The first target is a canary: a simulated controller that runs the real software, so a release that cannot even start is caught without touching a customer. The next step is a single real device on the bench. Only then the rest.

**A rollout is confirmed, not assumed.** After applying an update the controller reports back what it now runs. The rollout counts as done for that device only when the report arrives and the version matches. A device that goes silent after an update is the signal to stop, not something to notice a week later.

**Settings and data survive the update.** The package replaces the software. The configuration of the building and the recorded history live elsewhere and are not part of it. Separately, the controller keeps a periodic copy of its configuration on a second storage device and can restore itself after its main storage was reflashed.

**Order matters across components.** One release added a commissioning code that a new controller requires at first setup. If the controller release had gone out first, the published apps, which had no field for that code, could not have set up a new device at all. So the apps were released first, and the controller afterwards. The rule I kept from that: when two components change together, release first the one that still works with the old version of the other.

## What a release looks like

1. Tag the commit. Every published version has a tag, so "what exactly is running there?" has an answer.
2. Build from a clean checkout of that tag, never from a working folder with uncommitted changes.
3. Publish the package and its manifest to the update server. Publishing an already published version is refused.
4. Start the rollout for the canary, then for the bench device, and read the confirmation each time.
5. Check the device after the update with the same short list every time: all modules answering, control programs running, no restarts, the API responding.

## What I would keep

- A canary that is not a real customer device. It costs nothing to break.
- Confirmation from the device as the definition of "done".
- The same post-update checklist every time, so a difference stands out.
- Refusing to republish a version. If something is wrong, the fix gets a new number.

## What is still missing

Package signatures. The manifest protects against a corrupted or swapped download from the same server, but not against a compromised server. The verification code for signed manifests is written and waits for the signing keys to be created and stored properly; until then this is an open item, and I would not call the chain complete.
