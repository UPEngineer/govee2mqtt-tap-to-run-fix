# Run the Tap-to-Run fix in plain Docker

These steps are for people who run govee2mqtt in Docker instead of as a Home Assistant app
(Home Assistant Container, a NAS, a separate Linux server, and so on).
If you use Home Assistant OS, follow the app instructions in the [README](README.md) instead.

The image is `upengineer/govee2mqtt:tap-to-run-fix` on Docker Hub. It's govee2mqtt (upstream commit
`6f5a844`) with the fix for [wez/govee2mqtt#737](https://github.com/wez/govee2mqtt/issues/737) applied.
Use it only until an official govee2mqtt release includes the fix.

## Requirements

- **A Linux Docker host.** govee2mqtt needs `network_mode: host` to find your lights on the LAN.
  Docker Desktop on Windows and macOS doesn't fully support that.
- **An Intel/AMD (amd64) machine.** This image isn't built for ARM, so it won't run on a Raspberry Pi.
  On ARM, use the Home Assistant app in this repo, which compiles on your own machine.
- **An MQTT broker** that Home Assistant also uses, such as Mosquitto, with the MQTT integration set up in Home Assistant.
- Your **Govee account email and password**, and a **Govee API key** (request one in the Govee Home app under
  Profile → Settings → Apply for API Key).

## 1. Stop your current govee2mqtt

Only one govee2mqtt should run at a time. Two of them publish the same devices twice.

- **Existing Docker container:** run `docker compose down` in its folder, or `docker stop` the container.
- **Home Assistant app:** stop the *Govee to MQTT Bridge* app and turn off *Start on boot*.

Your Home Assistant entities keep their names, because this version publishes to the same MQTT topics.

## 2. Make a folder

```bash
mkdir -p ~/govee2mqtt/data && cd ~/govee2mqtt
```

The container runs as user ID 1000 and stores its cache in `data`. Give that user ownership of the folder:

```bash
sudo chown -R 1000:1000 data
```

**Don't skip this step.** If the container can't write to `data`, it crashes on startup over and over with
`failed to open "/data/govee2mqtt-cache.sqlite"`.

## 3. Create `docker-compose.yml`

```bash
nano docker-compose.yml
```

Paste this, then save (Ctrl+O, Enter, Ctrl+X):

```yaml
name: govee2mqtt
services:
  govee2mqtt:
    image: upengineer/govee2mqtt:tap-to-run-fix
    container_name: govee2mqtt
    restart: unless-stopped
    env_file:
      - .env
    # Required, so govee2mqtt can find your lights on the LAN
    network_mode: host
    volumes:
      - ./data:/data
```

## 4. Create `.env` with your settings

```bash
nano .env
```

Paste this and fill in your own values:

```ini
GOVEE_EMAIL=you@example.com
GOVEE_PASSWORD=your-govee-password
GOVEE_API_KEY=your-govee-api-key

# Your MQTT broker (often the Home Assistant machine's IP address)
GOVEE_MQTT_HOST=192.168.1.10
GOVEE_MQTT_PORT=1883
GOVEE_MQTT_USER=your-mqtt-user
GOVEE_MQTT_PASSWORD=your-mqtt-password

# C or F
GOVEE_TEMPERATURE_SCALE=F
# Your time zone, for log timestamps
TZ=America/Chicago
RUST_LOG_STYLE=always
# Uncomment for detailed logs if something goes wrong
#RUST_LOG=govee=trace
```

If you're switching from the Home Assistant app, use the same values as its **Configuration** tab.

The file holds passwords, so make it readable only by you:

```bash
chmod 600 .env
```

## 5. Start it

```bash
docker compose pull && docker compose up -d
```

Watch the logs (press Ctrl+C to stop watching; the container keeps running):

```bash
docker logs -f govee2mqtt
```

## What success looks like

- `Starting service`, followed by your Govee devices being listed.
- A line saying it connected to your MQTT broker.
- **No** `exec-plat/home ... 401 authorization token is invalid` or `Failed to parse one-clicks` lines.
- In Home Assistant, the `scene.govee_to_mqtt_one_click_*` scenes are available, including Tap-to-Runs you created recently.

After you add a new Tap-to-Run in the Govee app, press **Purge Caches** on the Govee2MQTT device in Home Assistant,
or run `docker restart govee2mqtt`, to load it.

## Troubleshooting

| What you see | What to do |
|---|---|
| `failed to open "/data/govee2mqtt-cache.sqlite"`, repeating | The `data` folder isn't writable. Run `docker compose down`, then `sudo chown -R 1000:1000 data`, then `docker compose up -d`. |
| `exec format error` | Your machine is ARM. Use the Home Assistant app in this repo instead. |
| MQTT connection refused or not authorized | Check `GOVEE_MQTT_HOST`, `GOVEE_MQTT_USER` and `GOVEE_MQTT_PASSWORD` in `.env`. |
| Lights that work over LAN are missing | Make sure `network_mode: host` is in the compose file and the host is on the same network as the lights. |
| Still getting the 401 on one-clicks | Check that the container is running this image: `docker inspect govee2mqtt --format '{{.Config.Image}}'`. |

## Going back to the official version

When a govee2mqtt release includes the fix, change the `image:` line to `ghcr.io/wez/govee2mqtt:latest`
and run `docker compose pull && docker compose up -d`. Or, if you used the Home Assistant app before,
run `docker compose down`, then update and start the official app and turn *Start on boot* back on.
