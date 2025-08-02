# ZMK Shield Cornix

This shield has been tested with Cornix using ZMK and provides full split-role configuration, battery power management, and Bluetooth central/peripheral setup per ZMK split guidelines 


![image](images/cornix_with_dongle.png)

## Supported Hardware: Cornix Split Keyboard

The **Cornix** is a low-profile, split ergonomic keyboard with the following compatible features:

- **Split, column‑staggered layout** (3×6 + thumb cluster layout).
- **Adjustable tenting support** at 10°, 18°, 25° (hardware‑based, no firmware hacks).
- **Kailh Choc V2 hot‑swap sockets** and support for LAK or LCK low‑profile keycaps.
- **Dual‑mode connectivity**: Wired USB‑C or Bluetooth wireless (left half as master).
- **Firmware**: Fully VIAL‑supported for keymaps and layer customization.
- Premium **CNC‑machined aluminum chassis**, custom damping foam, and portable storage pouch.

Cornix Split Tented Low‑Profile Ergo Keyboard (Jezail Funder)

### Overview
Cornix is a Corne‑inspired split ergonomic keyboard featuring a compact 3×6 column‑staggered layout with six thumb‑cluster keys (three per half). It offers adjustable tenting angles at 10°, 18°, and 25°, allowing users to reduce wrist strain and find a custom ergonomic alignment
.

### Key Features

    Aluminum CNC Case: Durable 6063 aluminum with sand‑blasted anodized finish available in Glacier Silver, Meteor Black, Aurora Purple, or Flame Red
    .

    Switch Compatibility: Hot‑swappable Kailh Choc V2 low‑profile switches (compatible with V2; older v1 may require slight modifications)
    .

    Keycap Options: Comes with either LAK PBT or LCK transparent PC low‑profile keycaps, depending on variant
    .

    Connectivity: Dual‑mode USB or Bluetooth (left half is master, right pairs as peripheral via BT; right‑side USB port is charging-only)
    .

    Customization: Full VIAL support for easy firmware remapping, macro setup, and layer configuration
    .

    Portable Design: Slim, split halves come with a custom storage bag, making it easy to pack and carry
    KeebFinder
    .

## Bootloader Recovery Instructions

The original RMK firmware removed the SoftDevice, so before flashing `zmk.uf2`, you need to restore the SoftDevice first. For specific steps, please refer to [bootloader/README.md](./bootloader/README.md).

## How to build Cornix Zmk firmware from scratch

This section will guide you through building the Cornix ZMK firmware from scratch using the official ZMK firmware development process.

### Prerequisites

Before starting, ensure you have the following:
- A GitHub account
- Git installed on your system
- Basic understanding of Git and GitHub
- Your Cornix keyboard PCBs ready

### Step 1: Initialize ZMK Config Repository

1. **Create a new repository** using the official ZMK config template:
   - Visit: https://github.com/zmkfirmware/unified-zmk-config-template
   - Click "Use this template" → "Create a new repository"
   - Name your repository (e.g., `cornix-zmk-config`)
   - Choose "Public" or "Private" as preferred
   - Click "Create repository"

2. **Clone your new repository locally**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
   cd YOUR_REPO_NAME
   ```

3. **Initialize ZMK development environment**:
   ```bash
   west init -l config/
   west update
   west zephyr-export
   ```

> **Important**: You should thoroughly read the ZMK documentation before proceeding, as ZMK firmware development has a learning curve.
> - ZMK Customization Guide: https://zmk.dev/docs/customization
> - ZMK Configuration: https://zmk.dev/docs/user-setup

### Step 2: Add Cornix Shield to Your Project

After initializing your zmk-config repository, follow the steps in the next section to integrate the Cornix shield.

## How to Add Cornix Shield to Existing ZMK Project

For users with existing zmk-config, add this repository dependency via west.yml and pull the latest version via west update:

### 1. Modify west.yml

Edit the `config/west.yml` file, add to the `manifest/remotes` section:

```yaml
remotes:
  - name: zmkfirmware
    url-base: https://github.com/zmkfirmware
  - name: cornix-shield
    url-base: https://github.com/hitsmaxft
```

Add to the `manifest/projects` section:

```yaml
projects:
  - name: zmk
    remote: zmkfirmware
    revision: main
    import: app/west.yml
  - name: zmk-shield-cornix
    remote: cornix-shield
    revision: main
```

### 2. Update Dependencies

```bash
west update
```

### 3. Configure Build

Edit the `build.yaml` file, add:

```yaml
include:
  - board: cornix_e73
    shield: cornix_left
    artifact-name: cornix_left

  - board: cornix_e73
    shield: cornix_right
    artifact-name: cornix_right
```

### 4. Build Firmware

Use your preferred method to build

### 5. Flash Firmware

Flash the generated `.uf2` files to the corresponding microcontroller:
- Left half: `build/left/zephyr/zmk.uf2`
- Right half: `build/right/zephyr/zmk.uf2`
