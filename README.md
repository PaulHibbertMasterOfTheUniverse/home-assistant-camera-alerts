# Home Assistant Camera Alerts

A reusable Home Assistant blueprint with snapshot notifications, optional video clips, and a zoomable clip player. No HACS integration required. Create one automation per camera.

**Starter release:** syntax checked, but not yet tested on a live Home Assistant installation or physical phone. This is based on a generic description, not an export of the original working system. Custom NVR buffered exports are not included.

## Install

1. Download this repository using **Code → Download ZIP** and unzip it.
2. Copy `blueprints/automation/camera_alerts/person_notification.yaml` into the same path under your Home Assistant configuration directory (usually `/config`). Alternatively, [import the blueprint](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FPaulHibbertMasterOfTheUniverse%2Fhome-assistant-camera-alerts%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fcamera_alerts%2Fperson_notification.yaml). Importing does not install the companion player or directories.
3. Create `/config/www/camera-alerts/`. Copy `www/camera-clip-player.html` into `/config/www/`.
4. Merge `examples/configuration.yaml` into your existing configuration, preserving existing entries. Check configuration and restart Home Assistant. Do not duplicate `homeassistant:` or `stream:` keys. If using a different config root, update the two filename paths in the blueprint and the allowlist.
5. Go to **Settings → Automations & scenes → Blueprints**, reload blueprints if needed, and create an automation from this blueprint.
6. Select your person/motion binary sensor and camera. Enter your full phone notification action, for example `notify.mobile_app_my_phone`, found under **Developer Tools → Actions**. Install/register the Companion App and enable phone notifications first.
7. Enter an existing live camera dashboard path, such as `/lovelace/cameras`. Use `examples/dashboard.yaml` as a manual card after replacing the camera entity.
8. Leave recording disabled for the first test. Let the sensor return to off, then cause a new detection. Confirm the snapshot notification and dashboard link work.
9. For clips, first test `camera.record` manually with a filename inside the allowed folder. Enable recording only if your camera supports it. The stream integration is required.

## How it works

Sensor turns on → optional continuous-on linger → snooze check → snapshot → phone notification linking to your live dashboard. With recording enabled, a later notification using the same tag links to that event clip. The player supports pinch zoom, mouse-wheel zoom, drag to pan, reset, playback controls and a Live camera link.

The clip link opens the player directly. An ordinary dashboard iframe does not automatically inherit the dashboard query parameter. The later notification waits for the recording action plus duration + 5 seconds; it can be delayed and is not proof that recording succeeded. Retry is available in the player. Notification replacement/re-alert behavior depends on phone settings.

## Privacy and retention

**Files under `/config/www` are exposed through `/local` without Home Assistant authentication. Anyone who can reach your server and knows a file URL can fetch it. Timestamped filenames are not access control. Use this edition only if that exposure suits your installation.**

Files accumulate. Set a retention process before leaving the automation running. Periodically delete old JPG and MP4 files only from `/config/www/camera-alerts/`, using your file manager or existing retention tool. Keep the player outside that directory. Deleted clips will no longer open from old notifications. Automatic cleanup is not included. Never upload camera footage, credentials or your personal Home Assistant backup to this repository.

## Snooze and cooldown

Create a Timer helper under **Settings → Devices & services → Helpers**. Select it in the blueprint and start it from its controls to snooze. Active, paused or unavailable selected timers suppress alerts; all selected timers must be idle. Leave the selection empty to disable snoozing.

Cooldown starts after all actions finish. New events during the run are ignored. Linger means the sensor stays on, not proof the person is physically present. Trigger waits do not survive Home Assistant restart or automation reload. Manual Run actions skips triggers and conditions, so it does not test snoozing.

## Buffering and compatibility

The portable clip path uses `camera.record`. Not all camera integrations support recording or snapshots. Lookback defaults to zero; nonzero lookback requires an already active HLS stream and actual length can vary. Recording begins after the first notification, so zero-lookback can miss the start of an event.

Frigate exports, vendor-specific recording APIs, RTSP capture and guaranteed pre-event buffering require a separate adapter based on the actual system. They are not implemented here.

## Troubleshooting and acceptance checks

- No snapshot: inspect the automation trace, directory, allowlist and camera snapshot support. Snapshot failure stops the run.
- No alert: test the phone action; check permissions, sensor off/on transition, linger, timer state and cooldown.
- Missing clip: inspect the trace and test camera.record directly. Recording errors are tolerated; a later link may still be sent after failure. Retry once recording has finished.
- Media only works on Wi-Fi: check Companion App server configuration and remote reachability. Do not expose another port just for the player.
- Before public use, test snapshot, live link, clip link, pinch zoom, pan, reset, playback controls, snooze expiry, repeated detections and missing clips on your own phone, locally and remotely.

## Validation and references

Blueprint and example YAML passed syntax parsing; all blueprint input references resolve. Player JavaScript passed syntax checking. Home Assistant runtime schemas, real camera behavior and phone compatibility remain untested. See `VALIDATION.md`.

- [Camera actions](https://www.home-assistant.io/integrations/camera/)
- [Recording and lookback](https://www.home-assistant.io/actions/camera.record/)
- [Blueprint schema](https://www.home-assistant.io/docs/blueprint/schema/)
- [Companion notifications](https://companion.home-assistant.io/docs/notifications/notifications-basic/)
- [Static-file authentication](https://www.home-assistant.io/integrations/http/)
