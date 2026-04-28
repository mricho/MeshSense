# MeshSense (mricho fork)

> **Heads up:** This is a personal fork of [Affirmatech/MeshSense](https://github.com/Affirmatech/MeshSense) with a few quality-of-life additions for antenna placement and link debugging. Everything in upstream MeshSense still works the same — this fork just adds extra capability on top. Upstream documentation and screenshots below remain accurate; only the items called out under **Fork-specific features** differ.

For the official, supported version of MeshSense, see [Affirmatech/MeshSense](https://github.com/Affirmatech/MeshSense).

---

## Fork-specific features

### 1. Lower minimum traceroute interval (1 min)

Upstream's "Traceroute Rate Limit (Minutes per Node)" setting was floored at **15 minutes**. In this fork the floor is **1 minute**, matching the firmware's actual rate limit. Useful when you're actively debugging a link and don't want to wait a quarter hour between probes.

Configurable in **Settings → Traceroute Rate Limit (Minutes per Node)**.

### 2. Route-back + per-hop SNR in the log

Meshtastic firmware has been reporting both directions of a traceroute (and per-hop SNR for each direction) for a while via `routeBack` / `snrTowards` / `snrBack`, but the upstream UI only renders the forward path. This fork shows the full info inline in the **Log** panel:

```
NBDY -(8.50dB)-> ?64ae0d19 -(5.25dB)-> !ac1cd197  |  back: !ac1cd197 -(6.00dB)-> ?64ae0d19 -(7.25dB)-> NBDY
```

SNR is shown in dB (firmware encodes it scaled ×4; the UI divides). Missing values are silently omitted. The 🔍 packet detail still has the full raw data if you want it.

### 3. Watch mode (continuous traceroute)

Next to the traceroute button (`↯`) on each node row there's a new **👁 button**. Toggling it on tells the API to traceroute that node continuously at the configured rate-limit interval (1 min if you've set the floor) until you turn it off.

- **One node at a time.** Toggling Watch on for a different node implicitly turns it off for the previous one.
- **Server-side timer.** Runs in the API process, so you can close the UI / put the laptop down and it keeps going. Stops on app exit (not persisted across restarts — by design, so a forgotten Watch doesn't keep hammering the network).
- **Skips when disconnected.** If the device disconnects, ticks are skipped (not queued up) and resume on reconnect.
- **Use case:** placing antennas. Mount it, walk to your radio, watch the dB values drift in real time as you adjust orientation.

API endpoint:

```
POST /watch  { "destination": <nodeNum> | null }
```

Posting `null` (or omitting `destination`) clears Watch.

---

# MeshSense (upstream)

**MeshSense** is a simple, [open-source](https://github.com/Affirmatech/MeshSense) application that monitors, maps and graphically displays all the vital stats of your area's Meshtastic network including connected nodes, signal reports, trace routes and more!

![](https://affirmatech.com/meshsense.png)

MeshSense directly connects to your Meshtastic node via Bluetooth or WiFi and continuously provides all the information you need to assess the health of your network. For more detailed information, take a peek at our [Frequently Asked Questions](https://affirmatech.com/meshsense/faq) or [Bluetooth Tips](https://affirmatech.com/meshsense/bluetooth).

## Headless Usage

To run MeshSense without a GUI, use the `--headless` flag. Additionally the `ACCESS_KEY` environment variable can be used to specify the privileged access key for remote connections to gain full permissions.

```sh
export ADDRESS=10.0.1.20  # Address of Meshtastic Node
export PORT=5920          # Port of remote interface

ACCESS_KEY=mySecretKey ./meshsense-x86_64.AppImage --headless

# Alternative execution:
dbus-run-session xvfb-run ./meshsense-arm64.AppImage --headless \
 --disable-gpu --in-process-gpu --disable-software-rasterizer
```

See also [Headless FAQ](https://affirmatech.com/meshsense/faq#headless)

## Debian Dependencies

Ubuntu and Raspberry Pi OS users will need the following dependency installed to run the AppImage:

```sh
sudo apt install libfuse2
```

To display unicode symbols on the buttons, it may be helpful to install `fonts-noto-color-emoji`

```sh
sudo apt install fonts-noto-color-emoji
```

## Development Setup

To run MeshSense from the source code, first clone this fork:

```sh
git clone --recurse-submodules https://github.com/mricho/MeshSense.git
cd MeshSense
```

Build `webbluetooth` Dependency.  Debian systems will need the `cmake` and `libdbus-1-dev` packages.

```
cd api/webbluetooth
npm i
npm run build:all
cd ../..
```

The `update.mjs` script will pull the latest code and install dependencies for the `ui`, `api`, and `electron` directories.

```sh
./update.mjs
```

During development, the electron portion is usually not needed. First start the UI Vite service as follows:

```sh
cd ui
PORT=5921 npm run dev
```

Leave the UI running and then also start the API service. The `DEV_UI_URL` will tell the API to forward any unhandled route requests to the UI service and should use the same port as above.

```sh
cd api
export DEV_UI_URL=http://localhost:5921
PORT=5920 npm run dev
```

The `PORT` variables in the above are optional and will default to the values in the example, but ensure `DEV_UI_URL` is present with the correct port if changed. These values may also be read from `.env` files `api/.env` and `ui/.env` respectively.

The front-end should now be accessible by connecting to the **API** service in a browser. Be careful not to connect to the UI service by accident. http://localhost:5920/

Any API changes will automatically reload the service. Any UI changes will be hot-reloaded by Vite.

**Please note:** currently certain event subscribers (particularly State variables) will duplicate their subscription when Vite hot-reloads resulting in duplicate events such as Log entries. Until this is fixed, the easiest solution is to refresh the browser to reset the events.

To build the `ui`, `api`, and `electron` components, the `build.mjs` script will accomplish this. The official electron builds are signed with an Affirmatech certificate on our build servers. The deployables will be placed in `api/dist` and `electron/dist`.
