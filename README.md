# PowerNap PCB, spin 1

First circuit board for the PowerNap watch: 32 x 40 mm, 4 layers, 66 parts placed by the factory.
Design notes and the reasons behind every choice: `docs/pcb-spin1-design.md`. The case: `case/README.md`.

**Read section 5 ("What has and has not been checked") before paying for anything.**

## 1. What is in this folder

| Path | What |
|---|---|
| `PowerNap_spin1.kicad_pro` | Open this in KiCad 10 to look at the schematic and the board. |
| `fab/PowerNap_spin1_gerbers.zip` | The board itself, for JLCPCB ("Gerber file"). |
| `fab/PowerNap_spin1_bom.csv` | The parts list with JLCPCB part numbers ("BOM file"). |
| `fab/PowerNap_spin1_cpl.csv` | Where each part goes and how it is turned ("CPL file"). |
| `fab/PowerNap_spin1_board.step` | 3D model of the board. |
| `out/PowerNap_spin1_schematic.pdf` | The schematic as a PDF. |
| `out/board_report.txt`, `out/stock.txt` | The last automatic board check and the last stock check. |
| `out/final_top.png`, `final_bottom.png`, `final_in1_ground.png`, `final_in2_3v3.png` | Pictures of the four copper layers. |
| `scripts/` | The programs that generate all of the above from one description (`design.py`). |
| `case/` | The 3D-printed case. |

Nothing here was drawn by hand. `scripts/design.py` lists every part and what each pin connects to;
`scripts/placement.py` says where each part sits. To change the design, change those and rebuild (section 6).

## 2. What the board does differently from the Arduino prototype

- It uses the same processor (nRF52840, inside the u-blox BMD-340 module) on the **same pins** as the
  Arduino Nano 33 BLE. So it runs the same firmware and is programmed from the Arduino IDE as a "Nano 33 BLE".
- Pulse sensor (MAX30101) on the **underside**, alone, so it can touch the skin through the case.
- Accelerometer BMI270 (the one all naps were recorded with). No magnetometer, no temperature sensor.
- USB-C for charging, uploading and the serial log. LiPo charger (100 mA) on board.
- On/off slide switch. Charging works in both positions.
- Buzzer (4 kHz) instead of the breadboard beeper; user button on the side; reset button behind a pin-hole.
- Status lights like the Arduino's: red/green/blue, the yellow "L" light, plus a red charging light.
- Five holes (`J2`) for the programmer, three test points (1.8 V, LED supply, system voltage).

## 3. Ordering from JLCPCB

Prices change; treat the numbers as rough. The quote on the website is the real one.

1. Go to jlcpcb.com, choose **Instant Quote / Order now**, and upload `fab/PowerNap_spin1_gerbers.zip`.
   It should detect **4 layers, 32 x 40 mm**. If it does not, stop and ask.
2. Board options: quantity **5**; thickness **1.6 mm**; surface finish **ENIG** (flat gold pads — worth the extra
   for the fine-pitch module and sensors); via covering **tented**; everything else as offered.
3. Switch on **PCB Assembly**. Choose **Standard** assembly, **both sides** (the pulse sensor is on the underside),
   quantity 2 or 5. Let JLCPCB add tooling holes and edge rails. Tick **"Confirm parts placement"**.
4. Upload `PowerNap_spin1_bom.csv` and `PowerNap_spin1_cpl.csv`.
5. Parts page: every line should be found and in stock (checked on 15 November 2026, see `out/stock.txt`).
   `J2` and `BT1` are not in the files on purpose — nothing is soldered there by the factory.
   If a part is out of stock, do not accept a suggested replacement without asking.
6. **Placement preview — check these before you pay** (section 5 explains why):
   - `U6` (pulse sensor, underside): its pin-1 dot must match the dot printed on the board.
   - `U1` (the big module): antenna end at the board edge, at the top.
   - `J1` (USB-C): opening pointing out over the right-hand edge.
   - The small chips `U2 U3 U4 U5 U7 U8 Q1 Q2`: pin-1 dot on the board's pin-1 mark.
   - `LED1`–`LED5`, `D1`, `D2`, `LS1` (buzzer, `+` mark).
7. Rough cost for 5 boards, 2 of them assembled: boards about $25 with ENIG, assembly set-up about $50,
   18 "extended" part types x $3 = $54, parts about $24 per board, shipping about $25, then Danish VAT (25 %)
   and a handling fee on top. Expect **roughly 1,500–2,200 DKK** in total. Not a quote.

## 4. First start-up (when the boards arrive)

You need an **nRF52840 DK** (Nordic's development board, about 400 DKK) once, to put the Arduino bootloader
into each new board. A new module is blank and does not show up on USB until this is done.

1. Look before you power: no solder bridges, pulse sensor straight in its place.
2. With a multimeter, check there is **no short** between `3V3` (hole 1 of `J2`, the square one) and `GND` (hole 5).
3. Plug in USB **without a battery**, switch on. Measure: test point `TP3` about 4.6 V; `J2` hole 1 = 3.3 V
   (3.25–3.35). If the 3.3 V is missing or the board gets warm, unplug.
   (The red charge light may glow or flicker with no battery connected. That is normal.)
4. Bootloader: connect the DK's **Debug out** pins to `J2` — hole 1 = 3.3 V (the DK's "VTG"), 2 = SWDIO,
   3 = SWDCLK, 4 = RESET, 5 = GND — and program this file, which is already on your PC:
   `C:\Users\pc\AppData\Local\Arduino15\packages\arduino\hardware\mbed_nano\4.6.0\bootloaders\nano33ble\bootloader.hex`
   with Nordic's "Programmer" app (part of nRF Connect for Desktop). I will walk you through this step when
   you get there.
5. Press reset twice quickly: the yellow light should pulse and a new COM port should appear. From here on
   the board is programmed from the Arduino IDE exactly like the Nano.
6. Firmware: PowerNap v11 with **one line changed**: `IMU.begin()` becomes `IMU.begin(BOSCH_ACCELEROMETER_ONLY)`
   (there is no magnetometer on this board, and the library otherwise waits for one).
   The new buzzer is a different type from the breadboard one, so the quiet "probe" beep (`PROBE_DUTY`)
   needs re-tuning by ear.
7. Only then solder the battery to `BT1` — **measure which wire is + first**. With the battery in, the red
   light is on while charging and goes out when full.

## 5. What has and has not been checked

Checked by program, every time the board is rebuilt:

- The schematic passes KiCad's electrical rule check with 0 messages, and its netlist is identical to `design.py`
  (every net, every pin).
- The board passes KiCad's design rule check with 0 errors (clearances, track widths, shorts, unconnected
  pins, parts overlapping) and matches the schematic part for part and pin for pin. See `out/board_report.txt`.
  What remains are 5 cosmetic notes about silkscreen lines running over the board edge at the USB socket
  and the switches; the factory simply cuts those off.
- No copper in the strip under the antenna. The ground-plane layer carries no tracks at all - it is one
  unbroken sheet (`out/final_in1_ground.png`).
- Every footprint was compared pad by pad with the footprint JLCPCB uses for the same part number, and the
  rotations in the placement file were calculated from that comparison (`scripts/check_rotations.py`).
- The finished placement file was then tested the way the factory uses it: JLCPCB's footprint for each of
  the 66 parts was placed at the file's position and rotation, and its pads compared with the board's.
  All land on their pads; the largest difference is 0.28 mm (`scripts/verify_cpl.py`).
- All 36 part types were in stock at JLCPCB on 15 November 2026.

Checked by me against documents:

- Every pin of every chip against its data sheet or a proven schematic (sources are listed at the top of `design.py`).
- The module's 68-pad footprint against the data sheet drawing and against JLCPCB's own footprint.

**Not checked — the real risks:**

1. **Nobody experienced has looked at this layout.** It was placed by coordinates and routed by programs
   (an automatic router, plus a small one I wrote for the last 22 connections). It obeys the rules, but rules
   do not catch everything (noise near the sensor, radio range, how well it solders). Known blemishes:
   about 37 mm of the 3.3 V and USB supply runs in 0.15 mm tracks instead of the usual 0.25 mm (still several
   times more than the current needs), and one small via sits right against a resistor pad (`R16`). For a first board that is normal, and it is why this is "spin 1" — but if you know
   anyone who designs boards, an hour of their time before ordering is the best money-saver there is.
2. **The underside part's rotation** (`U6`, the pulse sensor) follows the usual convention for bottom-side
   parts (mirrored, angle counted from below), which I could not confirm in JLCPCB's own documentation.
   If it were wrong the sensor would be soldered on turned by half a turn and would not work. That is why
   step 6 of the ordering list matters; JLCPCB's engineers also check pin-1 marks.
3. **The Arduino bootloader on this module** should work (same chip, same pins), but it has not been tried.
4. **The buzzer's loudness** inside the case, and the **radio range** with the battery beside the module,
   are unknown until a board exists.
5. Nothing has been built. A first board commonly needs one or two wire fixes; a second spin is the usual outcome.

## 6. Rebuilding after a change

Run from the project folder, in this order. `KP` means `"C:\Program Files\KiCad\10.0\bin\python.exe"`.

1. `py -3.11 hardware/spin1/scripts/gen_schematic.py` — writes the schematic
2. `py -3.11 hardware/spin1/scripts/verify_schematic.py` — must end with `SCHEMATIC OK`
3. `KP hardware/spin1/scripts/gen_board.py` — places the parts
4. `KP hardware/spin1/scripts/route_board.py` — routes (about 10 minutes). The automatic router does not give
   the same result twice; add `--reuse` to keep its last result and only redo the steps after it.
5. `KP hardware/spin1/scripts/verify_board.py` — must end with `BOARD OK`
6. `KP hardware/spin1/scripts/check_rotations.py` — only if a part or footprint changed
7. `py -3.11 hardware/spin1/scripts/export_fab.py` — writes `fab/`
8. `KP hardware/spin1/scripts/verify_cpl.py` — must end with `PLACEMENT FILE OK`
9. `py -3.11 hardware/spin1/scripts/check_stock.py` — live stock
10. `KP hardware/spin1/scripts/dump_bodies.py`, then Blender with `case/build_case.py` — rebuilds the case
