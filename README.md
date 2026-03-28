# tr-docker

Pre-built [Trunk Recorder](https://github.com/TrunkRecorder/trunk-recorder) Docker image with all common plugins included. Drop in as a replacement for a manual trunk-recorder build.

```bash
docker pull ghcr.io/trunk-reporter/trunk-recorder:latest
```

## Included Plugins

| Plugin | Source | Purpose |
|---|---|---|
| `mqtt_status` | [TrunkRecorder/tr-plugin-mqtt](https://github.com/TrunkRecorder/tr-plugin-mqtt) | Call/unit/recorder events over MQTT |
| `mqtt_dvcf` | [trunk-reporter/tr-plugin-dvcf](https://github.com/trunk-reporter/tr-plugin-dvcf) | Raw IMBE codec frames → DVCF files + MQTT (for IMBE-ASR) |
| `symbolstream` | [trunk-reporter/symbolstream](https://github.com/trunk-reporter/symbolstream) | Live codec frame streaming over TCP/UDP |
| `simplestream` | upstream (patched) | Audio streaming to remote consumers |
| `openmhz_uploader` | upstream | Upload calls to OpenMHz |
| `broadcastify_uploader` | upstream | Upload calls to Broadcastify |
| `unit_script` | upstream | Run scripts on unit events |

## Usage

```bash
docker run -d \
  --name trunk-recorder \
  --restart unless-stopped \
  --privileged \
  --device /dev/bus/usb:/dev/bus/usb \
  -v /var/run/dbus:/var/run/dbus \
  -v ./config.json:/app/config.json:ro \
  -v ./audio:/app/audio \
  ghcr.io/trunk-reporter/trunk-recorder:latest
```

Mount your existing `config.json` — everything else is the same as a standard trunk-recorder setup.

## Plugin Configuration

Add plugins to the `plugins` array in your `config.json`. See each plugin's README for full config options.

### mqtt_status (call/unit events)

```json
{
  "name": "mqtt_status",
  "library": "libmqtt_status_plugin",
  "broker": "tcp://YOUR_BROKER:1883",
  "topic": "tr/feeds",
  "unit_topic": "tr/units",
  "instanceId": "my-site"
}
```

### mqtt_dvcf (IMBE-ASR integration)

```json
{
  "name": "mqtt_dvcf",
  "library": "libmqtt_dvcf",
  "write_enabled": true,
  "mqtt_enabled": true,
  "broker": "tcp://YOUR_BROKER:1883",
  "topic": "tr/feeds"
}
```

### symbolstream (live streaming)

```json
{
  "name": "symbolstream",
  "library": "libsymbolstream",
  "streams": [
    {
      "address": "YOUR_TR_ENGINE_HOST",
      "port": 9090,
      "TGID": 0,
      "useTCP": true,
      "sendJSON": true
    }
  ]
}
```

## Full Stack

For the complete P25 transcription stack (tr-engine + tr-dashboard + imbe-asr + postgres + mosquitto), see [trunk-reporter/tr-stack](https://github.com/trunk-reporter/tr-stack).

## Notes

- Built from latest upstream trunk-recorder weekly + on every push
- `simplestream` includes a pending dangling-pointer fix ([TR PR #1107](https://github.com/TrunkRecorder/trunk-recorder/pull/1107)) that will be removed once merged upstream
- Requires `voice_codec_data()` API (merged in TR commit `c996900`) — already in this image
