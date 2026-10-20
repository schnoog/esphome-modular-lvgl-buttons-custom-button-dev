# lvgl_screenshot (vendored, PSRAM fix)

Copy of `esphome-modular-lvgl-buttons/components/lvgl_screenshot` with one more
fix for ESP-IDF builds without `CONFIG_SPIRAM_USE_MALLOC` (e.g. ESP32-P4
CrowPanel): the PNG encoder (stb_image_write) now allocates its ~w*h*3 bytes of
working memory in PSRAM (`STBIW_MALLOC` → `heap_caps_malloc_prefer`). Before,
plain `malloc()` failed silently in internal RAM, the PNG had 0 bytes and the
HTTP endpoint answered "Framebuffer unavailable". A failed encode is now logged.

Use with `path: draft/components` instead of `esphome-modular-lvgl-buttons/components`.

---


Serves a PNG of the LVGL framebuffer over HTTP — used for AI-assisted / visual
UI development without flashing hardware.

Vendored from **[agillis/esphome-lvgl-screenshot](https://github.com/agillis/esphome-lvgl-screenshot)**
(MIT, see LICENSE), with the LVGL 9 fix: `do_capture_()` now uses
`lv_snapshot_take()` (the v8 `disp->driver->draw_buf` / RGB565 bitfield API was
removed in LVGL 9), and `LV_USE_SNAPSHOT` is enabled via a build flag. Works on
ESPHome 2026.6+ (LVGL 9.5), host/SDL and ESP-IDF.

## Use

```yaml
external_components:
  - source:
      type: local
      path: esphome-modular-lvgl-buttons/components
    components: [lvgl_screenshot]

lvgl_screenshot:
  port: 8080
```

Then, with the (SDL or on-device) build running:

```bash
curl http://localhost:8080/screenshot > screenshot.png   # host/SDL
curl http://<device-ip>:8080/screenshot > screenshot.png # hardware
```

See [`AI_tools/SDL-lvgl-screenshot.yaml`](../../AI_tools/SDL-lvgl-screenshot.yaml)
for a full desktop harness.
