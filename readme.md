# Custom QMK Firmware for NuPhy Air96 V2 (ANSI)

This repository contains a modified version of the official NuPhy QMK firmware, optimized for custom status indications and complete compatibility with the **VIA Configurator (V3 API)**.

---

## 🚀 Key Improvements & Source Code

### 1. Per-Key RGB Status Indicators (`keymap.c`)
Moved the physical **CapsLock** and **NumLock** indicators away from the default sidebar patterns directly onto their respective mechanical keys using the advanced matrix rendering layer. 

The following snippet was appended to the absolute end of `keyboards/nuphy/air96_v2/ansi/keymaps/via/keymap.c`:

```c
#ifdef RGB_MATRIX_ENABLE
bool rgb_matrix_indicators_user(void) {
    // 1. Caps Lock (Forces solid Red backlight on CapsLock key when active)
    if (host_keyboard_led_state().caps_lock) {
        rgb_matrix_set_color(55, 255, 0, 0); 
    }

    // 2. Num Lock (Inverse Logic: Forces solid Red backlight on NumLock key when Numpad is OFF)
    if (!host_keyboard_led_state().num_lock) {
        rgb_matrix_set_color(33, 255, 0, 0); 
    }
    return true;
}
#endif
```

### 2. Intelligent Sidebar Connection Indicator (`side.c`)
Reprogrammed the left Side LED strip to function exclusively as a hardware connection status monitor, completely removing any flashing loops linked to CapsLock.

The original function `sys_led_show(void)` inside `keyboards/nuphy/air96_v2/ansi/side.c` was fully overwritten with the following logic:

```c
void sys_led_show(void)
{
    if (dev_info.link_mode == LINK_USB) {
        // 1. WIRED MODE -> Do nothing, let the standard custom RGB matrix animations run smoothly
    }
    else if (dev_info.link_mode == LINK_RF_24) {
        // 2. 2.4Ghz WIRELESS DONGLE -> Enforce static Orange color using active brightness level
        set_left_rgb(SIDE_BLINK_LIGHT, 127, 0x00);
    }
    else {
        // 3. BLUETOOTH MODE (Any active profile) -> Enforce static Blue color
        set_left_rgb(0x00, 0x00, SIDE_BLINK_LIGHT);
    }
}
```

---

## 🛠️ How to Compile and Flash

### 1. Build Command (QMK MSYS)
Ensure you are operating inside the dedicated `via` keymap folder to maintain complete VIA support and ProductID alignment:
```bash
qmk compile -kb nuphy/air96_v2/ansi -km via
```

### 2. Flashing to Hardware
1. Launch **QMK Toolbox**.
2. Select the compiled binary file: `nuphy_air96_v2_ansi_via.bin`.
3. Set the Microcontroller profile target to: `STM32F072`.
4. Power the keyboard to **Wired mode**, disconnect the cable, press and hold the physical **`Esc` key**, and plug the cable back in to trigger DFU bootloader mode.
5. Click **Flash**.

⚠️ **CRITICAL STEP:** Immediately after flashing completes, perform a hardware EEPROM reset by holding **`FN + [`** for 3 seconds. This forces the microcontroller to build fresh data maps required by VIA.

### 📥 3. Connecting to VIA Configurator
1. Open **[usevia.app](https://usevia.app)** in a compatible web browser.
2. Go to **Settings** (Gear icon) and toggle **Show Design tab**.
3. Open the **Design tab** (Paintbrush icon) and click **Load**.
4. Select the custom definitions template file located inside your local firmware workspace directory:
   `qmk_firmware/keyboards/nuphy/air96_v2/ansi/keymaps/via/NuPhy Air96 V2 via3.json`
5. Go back to the **Configure** tab and click **Authorize device** to manage your layouts.

---

## 🤖 Credits & Collaboration
All code modifications, logic routing enhancements (such as the inverse NumLock system), and QMK MSYS setup configurations within this repository were researched, implemented, and refined with the collaborative assistance of **Google Gemini AI**.
