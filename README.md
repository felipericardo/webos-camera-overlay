# webos-camera-overlay

Small live camera box shown over whatever is on a rooted LG webOS TV, meant for a doorbell or
alarm. App id `tld.my.camoverlay`. Built and tested on webOS 4.x (LG 65UK6530PSF).

## How it works

- **Window:** `appinfo.json` declares `"defaultWindowType": "overlay"` and
  `"transparent": true`. The compositor draws the app above the current one without
  changing the foreground app, the same mechanism LG uses for Settings and Google Assistant.
- **Image:** a 640x360 box in the top-right corner shows [go2rtc](https://github.com/AlexxIT/go2rtc)
  MJPEG (`stream.mjpeg`).
  - Images don't use the TV's single video output, so whatever is playing underneath keeps
    playing. A `<video>` here would black out live TV until it restarts.
  - A snapshot (`frame.jpeg`) fills the box while go2rtc starts the MJPEG transcode.
- **Closing:** after `timeout` seconds (default 60), or with Back, OK or Exit.

## Configure

```sh
cp app/config.example.js app/config.js
```

Set your go2rtc host and the default stream. `app/config.js` is git-ignored, and it is
packaged into the IPK on build.

The stream must have an MJPEG track. For an H.264/H.265 camera, give it its own go2rtc stream
that transcodes with ffmpeg:

```yaml
streams:
  front_door_mjpeg: ffmpeg:front_door_sub#video=mjpeg
```

go2rtc runs that transcode only while someone watches. Don't add the MJPEG track to a stream
that a recorder (e.g. Frigate) reads 24/7: the transcode then never stops.

## Launch parameters

| Param | Default | Meaning |
|---|---|---|
| `stream` | `stream` from `config.js` | go2rtc stream with an MJPEG track |
| `timeout` | `60` | seconds before closing |

```sh
# on the TV
luna-send -n 1 luna://com.webos.applicationManager/launch '{"id":"tld.my.camoverlay","params":{"stream":"front_door_mjpeg"}}'
```

## Build and install

Uses [webos-tools](https://github.com/felipericardo/webos-tools), checked out next to this repo.

```sh
python3 -I ../webos-tools/build_ipk.py app dist/tld.my.camoverlay_<version>_all.ipk
../webos-tools/install_ipk.sh <tv-ip> dist/tld.my.camoverlay_<version>_all.ipk --launch
```

Bump `version` in `app/appinfo.json` for every install. The TV must be rooted (Homebrew
Channel) with SSH as root.

## License

MIT, see [LICENSE](LICENSE).
