# DroneTM to Eve Converter

Converts a Potensic Atom 2 Drone Tasking Manager (DroneTM) flight plan into a `waypoints.json` that is compatible with Potensic Eve on iOS.

The tool runs in your browser. Your flight plan is processed on your own device.

> **Open the tool:** https://narendtt.github.io/dronetm-to-eve/

## What it does

- Reads the DroneTM flight plan (`.json`) and writes an Eve `waypoints.json`.
- DroneTM points that take a photo become Eve photo waypoints. Points without a photo become plain waypoints.
- The drone's heading follows the route.
- Shows the route on a map with Easting and Northing (WGS84 / UTM), with zoom and pan.
- Suggests a flight speed that keeps at least 2 seconds between photos, capped at 8 m/s (the Atom 2's top speed in normal mode).
- **Crosshatch** (optional) adds a second pass at 90° to the first, roughly doubling the photo points.
- **Oblique correction** (available only when the DroneTM plan uses a -45° gimbal pitch) re-spaces the photo points and moves the flight path back so a tilted camera photographs the area DroneTM planned. See [Oblique correction and GSD](#oblique-correction-and-gsd) below. **Show original route** draws the DroneTM route underneath for comparison.
- Warns when a plan has more than 200 waypoints. Potensic documents a limit of 200 waypoints and points of interest combined per Atom 2 mission.
- Saves the result as `waypoints.json`, and can also save a KML of the converted route for checking in Google Earth.

## What you need

- A Potensic Atom 2 and an iPhone or iPad with **Potensic Eve** installed.
- A computer with **Apple Devices** (the Apple Devices app on Mac or Windows) and a USB cable. Apple Devices is how the files are copied to and from Eve, so it is required.
- A web browser.

## How to use it

### 1. Create your flight plan in Drone Tasking Manager

1. Go to the Drone Tasking Manager (DroneTM) website and create your mapping flight plan for the **Potensic Atom 2**.
2. Draw the area you want to map and choose your flight settings (GSD, overlaps and gimbal angle). Note the GSD, because Oblique correction needs it.
3. Download the flight plan as a `.json` file and keep it somewhere you can find it.

### 2. Create a dummy mission in Eve on iOS

Eve has to create a `waypoints.json` before one can be replaced.

1. Open **Potensic Eve** on your iPhone or iPad.
2. Create a simple waypoint mission with a few waypoints and save it. It can be anything; it will be overwritten.

### 3. Copy the TrajectoryFlight folder to your computer

1. Connect your iPhone or iPad to your computer with a cable and open **Apple Devices**.
2. Select your device, then choose **Files**.
3. Find **Potensic Eve** in the list and open it.
4. Drag the **TrajectoryFlight** folder to your computer.
5. Keep a second copy of the folder as a backup.

### 4. Convert the flight plan

1. Open the tool (link above).
2. Load your DroneTM `.json` file by dropping it on the page or choosing it.
3. Check the settings:
   - **Speed**: use the suggested speed, or press **Use this speed**.
   - **Height**: keep the heights from your DroneTM plan, or set a fixed height.
   - **Crosshatch**: tick it to add a second pass at 90°.
   - **Oblique correction**: tick it if your plan is at -45°. It is greyed out for any other angle. Then enter the **GSD** you set in DroneTM.
4. Look over the route on the map. Tick **Show original route** (Oblique correction only) to see the DroneTM route underneath. If a warning appears about more than 200 waypoints, reduce the area or turn off Crosshatch, because Potensic documents a limit of 200 waypoints per mission.
5. Click **Save waypoints.json**. If saving is not available in your browser, use **Copy JSON** and paste the text into a new file named `waypoints.json`.

### 5. Overwrite waypoints.json in the TrajectoryFlight folder

1. Open the **TrajectoryFlight** folder you copied in step 3.
2. Find the dummy `waypoints.json` that Eve created. It may be inside a subfolder.
3. Replace it with the `waypoints.json` you just saved. Keep the file name exactly `waypoints.json`.

### 6. Upload the folder back to Eve

1. Reconnect the device and open **Apple Devices** > **Files** > **Potensic Eve**.
2. Drag the edited **TrajectoryFlight** folder into Potensic Eve.
3. When asked, choose to replace the existing files.

### 7. Check the mission in Eve and fly

1. Open **Potensic Eve** and load the mission.
2. Check the route, waypoint count, height and speed.
3. Fly only when conditions are safe and legal, and follow your local drone regulations.

## Oblique correction and GSD

A camera tilted 45° looks at the ground ahead of the drone, not below it, so photos taken at DroneTM's points would miss the start of the area being mapped. Oblique correction fixes this in three ways:

- **Path moved back.** Each pass is moved back along its own direction of travel, so the centre of each photo lands on the point DroneTM planned. The bearing of the photo lines does not change.
- **Photos re-spaced.** Photo spacing and the distance between passes are recalculated from the camera's whole-frame footprint (Atom 2: 79.4° diagonal field of view, 4:3), aiming for 75% front and side overlap, and no more than 80%.
- **Heights and the GSD.** DroneTM heights are measured from the take-off point and do not show how high the drone is above the ground it photographs. If you take off from low ground to map a high hill, the file's heights are much larger than the real height above the ground. The tool therefore asks for the **GSD** from your DroneTM plan:
  1. DroneTM's GSD is for a camera pointing straight down. The tool converts it to a height above the ground (the Atom 2 has a 4000 px wide image; 2.0 cm/px is about 60 m).
  2. A camera tilted 45° sees the ground from farther away, so its GSD is coarser. To keep the GSD **across the view** equal to the GSD you entered, the drone flies at that height × sin 45° (about 42.6 m for 2.0 cm/px). The camera is then the same distance from the ground as DroneTM's camera. The GSD **along the view** is the entered GSD ÷ sin 45° (2.83 cm/px for 2.0 cm/px), which is unavoidable foreshortening.
  3. Every waypoint height in `waypoints.json` is lowered by the difference, so the route keeps the same height above the ground that DroneTM planned, less that difference. The tool shows a warning with the amount. **Check that the terrain and obstacles under the route leave enough clearance before flying.**

If you leave the GSD empty, the tool treats the heights in your file as the height above the ground. That is only right when the ground is level with the take-off point, and the GSD of your photos will not match the GSD you planned, so entering the GSD is recommended.

Things to know:

- The GSD values hold at the centre of the frame. Towards the far edge of a tilted view the ground is much coarser.
- The correction assumes level ground between the drone and the centre of the frame. On a slope the photos land a little short (uphill) or long (downhill).
- Lowering the route puts more photo points and passes in the plan, so check the waypoint count against the 200 limit.

## Notes

- **Est. flight** in the tool is distance divided by speed, so it is a minimum.
- **Save KML** downloads a zip containing `waypoints.kml`. Unzip it, or rename the zip to `.kmz`, to open it in Google Earth. Positions are rounded to the nearest centimetre in Easting and Northing, so they can differ from `waypoints.json` by under 1 cm. (The browser's save dialog does not accept `.kml` files directly, which is why it is zipped.)
- If you see a height or camera angle in Eve that does not look right, do not fly the mission. Check the settings in the tool and test with a small plan first.

## Disclaimer

This is an unofficial tool and is not affiliated with Potensic, Drone Tasking Manager or Apple. The Eve `waypoints.json` format is not published, so the fields were worked out from sample files. Always check a mission in Eve before flying.
