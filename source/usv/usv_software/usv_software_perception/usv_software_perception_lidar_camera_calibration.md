# Camera and LiDAR calibration

> Author: Pedro Librado

This page tells you how to calibrate the USB webcam (Logitech C920) and the
Velodyne VLP-16 LiDAR of the USV. It also tells you how to find and repair the
usual problems. The scripts and the configuration files are in the
[visionsystemx](https://github.com/vanttec/visionsystemx) repository. The text
uses ASD-STE100 (Simplified Technical English).

```{contents}
:local:
:depth: 2
```

---

## 1. Purpose

The boat does not use the ZED camera now. It uses a USB webcam for the image
and the VLP-16 for the distance. The software must know two sets of values:

| Set | What it is | File |
|---|---|---|
| Intrinsic values | The lens values of the camera: focal length (fx, fy), center point (cx, cy) and distortion | `config/webcam_calibration.yaml` and the `camera` section of `config/fusion.yaml` |
| Extrinsic values | The position and the rotation of the LiDAR relative to the camera (a 4 × 4 matrix) | `extrinsic_matrix` in `config/fusion.yaml` |

Do the intrinsic calibration first. The extrinsic calibration uses the
intrinsic values.

**CAUTION:** The calibration is correct only while the camera and the LiDAR
stay in the same position on their mount. If one of them moves, do the two
calibrations again. A clip mount can move the camera. Attach the camera with a
screw or with glue, and put a mark on its position.

**CAUTION:** Use the same camera, the same resolution (1280 × 720) and the
same focus as on the boat. A different focus changes the focal length.

---

## 2. How the system uses the calibration

`launch/vision.launch.py` starts these nodes:

| Node | Function |
|---|---|
| `usb_cam` | Reads the webcam. Publishes `/bebblebrox/video` and `/bebblebrox/video/camera_info`. Reads the intrinsic values from `camera_info_url` |
| Focus lock (`v4l2-ctl`) | 3 s after `usb_cam` starts, it stops the autofocus and sets the focus to infinity |
| Velodyne driver | Publishes `/velodyne_points` |
| `yolo` | Finds objects in the image. Publishes `/yolo/detections` |
| `lidar_bbox_range` | Puts the LiDAR points into the image with the extrinsic values. Gives the distance of each detection on `/bebblebrox/objects/yolo`. Publishes a test image on `/bebblebrox/debug/frustum` |

The calibration tools are in `scripts/`:

| Script | Function |
|---|---|
| `collect_calib_pairs.py` | You click the board in rviz and in the photo. The script calculates the board center from the LiDAR rings and writes one point pair per click |
| `calibrate_lidar_camera.py` | Calculates the extrinsic matrix from the point pairs (OpenCV `solvePnP`) |

---

## 3. Equipment and software

### 3.1 Equipment

- The camera and the LiDAR, attached to their final mount.
- The VLP-16 interface box and its power supply (9–18 V DC).
- An Ethernet cable from the interface box to the computer.
- A printed checkerboard on a flat, rigid surface. Measure the side of one
  square with a ruler.
- A flat, rigid board for the LiDAR calibration. The team board is MDF,
  70 × 61 cm. Put a cross of tape at its center.
- A tape measure.
- A second person to hold the board.

### 3.2 Software

You can use the Jetson or a laptop. A laptop is easier: rviz is faster and you
can do Procedure C at home with the bag.

Use ROS 2 Humble (native or in a container) with these packages:

```bash
sudo apt install ros-humble-velodyne ros-humble-usb-cam ros-humble-image-transport-plugins ros-humble-camera-calibration v4l-utils
```

If you use the team dev container, add these packages to the `Dockerfile` as a
separate `RUN apt-get install` layer at the end. Then the packages stay in
the image when you make the container again.

In this document, `$WS` is your ROS 2 workspace root (in the dev container it
is `/ws`), and `$VSX` is `$WS/src/vanttec_usv/src/visionsystemx`.

---

## 4. One-time setup

### 4.1 Python environment

**NOTE:** Some environments have NumPy 2 and an OpenCV without window
support. The ZED container and the Jetson have this problem. ROS Humble
(`cv_bridge`) needs NumPy 1, and the calibration tools need windows.

1. Make sure that the next command shows NumPy 1.x and an OpenCV that opens
   windows:

   ```bash
   PYTHONPATH=/usr/lib/python3/dist-packages:$PYTHONPATH python3 -c "import numpy, cv2; print(numpy.__version__, cv2.__version__)"
   ```

2. In this document, the commands that need windows or `cv_bridge` start with
   `PYTHONPATH=/usr/lib/python3/dist-packages:$PYTHONPATH`. Do not set this
   value for all the system. Other tools (for example `pyzed`) need NumPy 2.

3. If you use SSH, connect with `ssh -Y`. Do this for each terminal that opens
   a window. Make sure that `echo $DISPLAY` shows a value.

### 4.2 Network for the VLP-16

The VLP-16 has the address `192.168.1.201`. It sends its data to all the
computers on the network (`255.255.255.255`, UDP port 2368).

Do these steps on the host computer, not in a container:

1. Find the name of the Ethernet port:

   ```bash
   ip -br link
   ```

2. Make a NetworkManager profile with a fixed address. Use your port name in
   place of `enp130s0`:

   ```bash
   sudo nmcli con add type ethernet ifname enp130s0 con-name velodyne ipv4.method manual ipv4.addresses 192.168.1.100/24 ipv6.method ignore
   ```

   ```bash
   sudo nmcli con up velodyne
   ```

   **NOTE:** Do not use `ip addr add`. NetworkManager removes a manual
   address after some time.

3. If the firewall (`ufw`) is on, open it for the port:

   ```bash
   sudo ufw allow in on enp130s0
   ```

4. Make sure that LiDAR packets arrive. You must see 3 lines from
   `192.168.1.201`:

   ```bash
   sudo tcpdump -i enp130s0 -n -c 3 udp port 2368
   ```

### 4.3 ROS isolation

If the Jetson and your computer are on the same network with the same
`ROS_DOMAIN_ID`, the topics of the two computers mix. In each terminal of
your computer, do this command:

```bash
export ROS_LOCALHOST_ONLY=1
```

---

## 5. Procedure A: intrinsic calibration

Time: approximately 15 minutes.

### 5.1 Start the camera

1. Find the device of the webcam. Laptops have more cameras (for example an
   infrared camera):

   ```bash
   v4l2-ctl --list-devices
   ```

   Use the first `/dev/videoN` under the C920 name. In the next steps, this
   document uses `/dev/video4`.

2. Start the camera:

   ```bash
   ros2 run usb_cam usb_cam_node_exe --ros-args -p video_device:=/dev/video4 -p pixel_format:=mjpeg2rgb -p image_width:=1280 -p image_height:=720 -p framerate:=30.0 -r image_raw:=/bebblebrox/video
   ```

   **NOTE:** The warning `Camera calibration file ... not found` is normal
   here.

3. In a second terminal, stop the autofocus. Do the two commands in this
   sequence:

   ```bash
   v4l2-ctl -d /dev/video4 -c focus_automatic_continuous=0
   ```

   ```bash
   v4l2-ctl -d /dev/video4 -c focus_absolute=0
   ```

4. Make sure that the two values are 0:

   ```bash
   v4l2-ctl -d /dev/video4 -l | grep focus
   ```

**CAUTION:** The camera starts with autofocus each time that you connect it.
Do steps 3 and 4 again after each connection.

### 5.2 Calibrate

1. Count the inner corners of the checkerboard. An inner corner is a point
   where four squares touch. A board with 6 × 8 squares has 5 × 7 inner
   corners.

2. Start the calibrator. Use your inner corners for `--size` and the square
   side in meters for `--square`:

   ```bash
   PYTHONPATH=/usr/lib/python3/dist-packages:$PYTHONPATH ros2 run camera_calibration cameracalibrator --size 5x7 --square 0.028 --ros-args -r image:=/bebblebrox/video
   ```

3. Set the slider "Camera type" to **0 (pinhole)**.

4. Hold the board 1.5–3 m from the camera. The focus is at infinity, thus a
   near board is not clear.

5. Move the board slowly to all areas of the image: the corners, the edges
   and the center. Turn it to the left, to the right, up and down. Move it
   near and far.

   When the calibrator finds the board, it shows colored lines on the
   corners. The bars X, Y, Size and Skew become longer.

6. Stop at 40–60 samples. When the four bars are green, click
   **CALIBRATE** one time.

   **NOTE:** The window does not respond during the calculation. This is
   normal. Do not click again. The calculation is complete when the terminal
   shows `D = ...` and `K = ...`.

7. Click **SAVE** one time. Click **COMMIT** never.

8. In the same terminal environment (for example, in the container), extract
   the result:

   ```bash
   cd /tmp && tar xzf calibrationdata.tar.gz ost.yaml && cat ost.yaml
   ```

### 5.3 Check and install the result

1. Make sure that the values are correct:

   | Value | Correct result |
   |---|---|
   | `image_width`, `image_height` | 1280, 720 |
   | fx and fy | Difference less than 2 % |
   | cx, cy | Near 640, 360 |

2. Copy the file into the package and set the camera name that `usb_cam`
   uses:

   ```bash
   cp /tmp/ost.yaml $VSX/config/webcam_calibration.yaml && sed -i 's/^camera_name: .*/camera_name: default_cam/' $VSX/config/webcam_calibration.yaml
   ```

3. Copy `camera_matrix` and `distortion_coefficients` into the `camera`
   section of `config/fusion.yaml` (`camera_matrix` and `dist_coeffs`).

---

## 6. Procedure B: record the calibration bag

Time: approximately 20 minutes. This is the only procedure that needs the
equipment.

### 6.1 Start the sensors

1. Do the network check (section 4.2, step 4).

2. Terminal 1, start the LiDAR:

   ```bash
   ros2 launch velodyne velodyne-all-nodes-VLP16-launch.py
   ```

3. Terminal 2, start the camera (section 5.1, step 2).

4. Terminal 3, lock the focus (section 5.1, steps 3 and 4).

5. Terminal 3, record a 5 s test bag:

   ```bash
   cd /tmp && rm -rf prueba && timeout -s INT 5 ros2 bag record /velodyne_points /image_raw/compressed -o prueba; ros2 bag info prueba | grep Count; rm -rf prueba
   ```

   The two topics must have a count more than 0. If `/velodyne_points` is 0,
   do not continue. Refer to section 10.4.

   **NOTE:** The counts are less than 5 s × the rate, because the recorder
   needs approximately 3 s to find the topics.

   **NOTE:** Do not use `ros2 topic hz` for the point cloud. It shows a rate
   that is too low.

### 6.2 Board positions

The script finds the board center from the LiDAR rings. For a good result:

- Turn the board approximately 45° (a diamond shape). If the board is
  straight, the script cannot find the height of its center.
- The board must be flat and vertical, and it must face the camera.
- The holder must hold the board at its side or at its bottom edge.

  **CAUTION:** Do not hold the board against the body. The body is then in
  the same plane as the board, and the result is not correct.

- Hold each position for **5 seconds**, then move to the next position.
- The full board must be in the image.
- At least 2 LiDAR rings must touch the board. A far or low board can get
  only 1 ring.

Do 15–20 positions with this pattern:

| | Left | Center | Right |
|---|---|---|---|
| Near (3–5 m), high | ✓ | ✓ | ✓ |
| Near (3–5 m), low | ✓ | ✓ | ✓ |
| Far (7–10 m), high | ✓ | ✓ | ✓ |
| Far (7–10 m), low | ✓ | ✓ | ✓ |

"High" is above the head. "Low" is at the waist. The VLP-16 sees only ±15°
up and down. Thus, at 3–5 m, keep the board within approximately 1 m of the
LiDAR height.

### 6.3 Record

1. Measure the distance from the center of the camera lens to the center of
   the LiDAR with a tape measure. The LiDAR origin is on its axis,
   approximately 3.8 cm above its base. Write down the values. You use them
   to check the result.

2. Start the recording:

   ```bash
   cd $WS && ros2 bag record /velodyne_points /image_raw/compressed -o calib_bag
   ```

3. Wait for `All requested topics are subscribed`. Then do the positions.

4. Push Ctrl+C to stop.

5. Make sure that the two topics have data:

   ```bash
   ros2 bag info $WS/calib_bag | grep Count
   ```

   Expected: approximately 10 clouds per second and 24–30 images per second.

6. Disconnect the equipment only after this check.

---

## 7. Procedure C: collect the point pairs

Time: approximately 30 minutes. You do not need the equipment.

**NOTE:** Do not start the LiDAR driver or `usb_cam` during this procedure.
They mix with the bag data.

### 7.1 Start the tools

1. Terminal 1, play the bag. It starts paused:

   ```bash
   ros2 bag play $WS/calib_bag --start-paused
   ```

2. Terminal 2, start rviz:

   ```bash
   rviz2
   ```

3. In rviz, do these settings:
   - **Fixed Frame**: `velodyne`.
   - **Add → By topic → `/velodyne_points` → PointCloud2**. Set Size to
     0.05.
   - **Add → By display type → PointCloud2**, topic `/calib/board_points`.
     Set a fixed color and Size 0.08. It shows the points that the script
     uses.
   - **Add → By display type → PointStamped**, topic
     `/calib/board_center`. Set Radius 0.1.

4. Optional: to see the camera image in rviz, start this node and add an
   **Image** display for `/bebblebrox/video`:

   ```bash
   ros2 run image_transport republish compressed raw --ros-args -r in/compressed:=/image_raw/compressed -r out:=/bebblebrox/video
   ```

5. Terminal 3, start the script. Use the size of your board in meters:

   ```bash
   PYTHONPATH=/usr/lib/python3/dist-packages:$PYTHONPATH python3 $VSX/scripts/collect_calib_pairs.py --ros-args -p board_w:=0.70 -p board_h:=0.61 -p image_topic:=/image_raw/compressed -p out_dir:=$WS/calib_pairs
   ```

   The script must show `ready: board edges 0.7x0.61m`.

### 7.2 Collect one pair

Do these steps for each board position:

1. Click in terminal 1 to give it the keyboard focus.
2. Push the **space bar** to play. When the board is stopped in a new
   position, push the **space bar** again to pause.
3. In rviz, select **Publish Point** (or push P). Click on the board in the
   point cloud.
4. Read the message in terminal 3:

   | Message | Meaning | Action |
   |---|---|---|
   | `board center ... rms=X cm`, X < 2.5 | Good fit | Continue to step 5 |
   | `rms` more than 2.5 cm | A different object is in the fit | Push Esc in the photo window. Click a different area of the board |
   | `need >=2 rings` | Only 1 ring touches the board | Go to the next position |
   | `closest image is +X s from the cloud` | No photo near the time of the cloud | Push Esc. Click again |

5. A photo window opens. It can be behind other windows. Click on the board
   center (the tape cross). You can click again to move the cross.
6. Push **Enter** to save. Push **Esc** to discard.
7. Make sure that terminal 3 shows `saved posNN`.

**NOTE:** Publish Point stops after each click. Select it again for each
pair.

**NOTE:** To go back in the bag, stop terminal 1 with Ctrl+C. Then start it
again with `--start-offset <seconds>`. The script continues to add pairs to
the same file.

**NOTE:** To start the pairs again, stop the script and delete
`$WS/calib_pairs`.

### 7.3 Result

The script writes:

- `$WS/calib_pairs/corr.tsv`: one comment line and one line `u v x y z` for
  each pair. The script writes each line to the disk immediately.
- `$WS/calib_pairs/posNN.png`: the photo of each pair.

---

## 8. Procedure D: calculate the extrinsic values

1. Do the calculation:

   ```bash
   cd $VSX && python3 scripts/calibrate_lidar_camera.py --intrinsics config/webcam_calibration.yaml --corr $WS/calib_pairs/corr.tsv
   ```

   **CAUTION:** Do not use `--fusion-out`. It writes the correct values, but
   it removes all the comments from `fusion.yaml`.

2. Make sure that the result is correct:

   | Check | Limit |
   |---|---|
   | `mean inlier err` | Less than 3 px |
   | Inliers | At least 80 % of the pairs |
   | Last column of the matrix (tx, ty, tz) | Must agree with the tape measure. This is the LiDAR position in camera axes: x right, y down, z forward |

3. If some pairs are `OUT` with a large error, open `corr.tsv`. Delete those
   lines. Do step 1 again.

4. If the mean error stays more than 3 px, or the translation does not agree
   with the tape measure, refer to section 10.6.

5. Copy the matrix into `extrinsic_matrix` in `config/fusion.yaml`. Keep the
   format and the comments of the file.

6. Commit `config/webcam_calibration.yaml` and `config/fusion.yaml`. Keep
   `corr.tsv` outside the repository, or attach it to the pull request.

---

## 9. Procedure E: install and test on the Jetson

1. Get the new files and build:

   ```bash
   cd ~/vanttec_usv/src/visionsystemx && git checkout main && git pull
   ```

   ```bash
   cd ~/vanttec_usv && colcon build --packages-select visionsystemx && source install/setup.bash
   ```

2. Make sure that `v4l2-ctl` is installed. If not, install `v4l-utils`.

3. Find the webcam device with `v4l2-ctl --list-devices`. If it is not
   `/dev/video0`, change `video_device` in `launch/vision.launch.py`.

4. Make sure that `camera_info_url` in `launch/vision.launch.py` points to
   `config/webcam_calibration.yaml` on the Jetson.

5. Start the system:

   ```bash
   ros2 launch visionsystemx vision.launch.py
   ```

6. Make sure that the intrinsic values are loaded. `k` must start with
   931.46 (not 0):

   ```bash
   ros2 topic echo /bebblebrox/video/camera_info --once
   ```

7. Look at `/bebblebrox/debug/frustum` (with `rqt_image_view` or
   `web_video_server`). The LiDAR points must be on the objects.

8. Put a buoy at a known distance (5, 10 and 15 m). Compare the distance with
   `x` in `/bebblebrox/objects/yolo`. The difference must be less than
   approximately 20 cm.

---

## 10. Troubleshooting

### 10.1 Python and display

| Symptom | Cause | Remedy |
|---|---|---|
| `AttributeError: _ARRAY_API not found` or `numpy.core.multiarray failed to import` | NumPy 2 is installed, but OpenCV or `cv_bridge` needs NumPy 1 | Put `PYTHONPATH=/usr/lib/python3/dist-packages:$PYTHONPATH` before the command. On the Jetson, you can also install `numpy<2` in `~/.local` (do `pip3 check` after) |
| `The function is not implemented. Rebuild the library with ... GTK` | This OpenCV has no window support (for example the CUDA build on the Jetson) | Use the `PYTHONPATH` value above. It selects the apt OpenCV |
| `Can't initialize GTK backend` or `could not connect to display` | The terminal has no display | Connect with `ssh -Y`. Make sure that `echo $DISPLAY` shows a value |
| `$DISPLAY` is empty after `ssh -Y` | The SSH server does not forward X11 | On the Jetson: install `xauth`, set `X11Forwarding yes` in `/etc/ssh/sshd_config`, restart `ssh` |
| rviz stops with an OpenGL error over SSH | No GPU rendering through SSH | Start it with `LIBGL_ALWAYS_SOFTWARE=1 rviz2` |
| `ros2: error: unrecognized arguments` | `ros2 topic hz` accepts only one topic | Do the command one time for each topic |
| `error: no such option: --image:` | ROS 1 syntax | Use `--ros-args -r image:=/topic` |

### 10.2 Camera

| Symptom | Cause | Remedy |
|---|---|---|
| `mjpeg2rgb is unsupported by the selected device` | Wrong device. Laptops have an infrared camera (8-bit grey, 640 × 360) | Find the C920 with `v4l2-ctl --list-devices` |
| `Permission denied` on `focus_absolute` | You cannot change `focus_absolute` while autofocus is on | Send `focus_automatic_continuous=0` first, then `focus_absolute=0`, as two commands |
| `unknown control 'focus_auto'` in the `usb_cam` log | `usb_cam` uses an old control name. The autofocus stays on | `vision.launch.py` locks the focus with `v4l2-ctl`. For a manual start, lock it yourself (section 5.1) |
| `/bebblebrox/video/camera_info` has `k` with all 0 | `usb_cam` did not load a calibration file | Use `vision.launch.py`, or give `camera_info_url`. Stop other `usb_cam` nodes |
| `Camera calibration file ... default_cam.yaml not found` | No `camera_info_url` | Not a problem for recording or for intrinsic calibration |
| `/bebblebrox/video/compressed` does not exist | A manual `usb_cam` with a remap publishes `/image_raw/compressed` | Use `/image_raw/compressed`. `vision.launch.py` publishes `/bebblebrox/video/compressed` |
| The camera rate is 24 fps, not 30 | Low light: the camera uses a longer exposure | Not a problem for the calibration |

### 10.3 Intrinsic calibrator

| Symptom | Cause | Remedy |
|---|---|---|
| `Package 'camera_calibration' not found` | The package is not installed | `sudo apt install ros-humble-camera-calibration`. Open a new terminal |
| The image is grey and no lines show | The calibrator uses a grey image (normal). The board is not found | Make sure that `--size` is the number of inner corners. Hold the board 1.5–3 m away so that it is clear. Show the full board with its white border |
| Camera type shows 1 (fisheye) | Default slider position | Set it to 0 (pinhole) |
| The window does not respond after CALIBRATE | The calculation stops the window (normal) | Wait. Do not click again. Use fewer samples next time (40–60) |
| SAVE does not become blue | The window was frozen, or a second CALIBRATE started | Wait for the second calculation. The values in the terminal are also correct |
| `tar: calibrationdata.tar.gz: Cannot open` | The file is in `/tmp` of the environment that ran the calibrator (for example the container) | Do the `tar` command in that environment |

### 10.4 LiDAR and network

| Symptom | Cause | Remedy |
|---|---|---|
| `tcpdump` shows nothing | No power, no cable, or wrong port | Check the power LED, the cable and the port name |
| `sudo: tcpdump: command not found` | You are in the container | Do the command on the host computer |
| `tcpdump` shows packets, but `/velodyne_points` has no data | The port has no IPv4 address, or `ufw` stops the packets | Make the NetworkManager profile (section 4.2). Do `sudo ufw allow in on <port>` |
| The address goes away after some time | NetworkManager removes manual addresses | Use the `nmcli` profile, not `ip addr add` |
| `ros2 topic hz /velodyne_points` shows approximately 3 Hz | The Python tool is slow for large clouds | Do a 5 s test bag (section 6.1). Expected: 10 Hz |
| Small `RcvbufErrors` count in `/proc/net/snmp` | The UDP receive buffer is small | Optional: `sudo sysctl -w net.core.rmem_max=26214400 net.core.rmem_default=26214400` |
| The bag has 0 clouds | The LiDAR was disconnected or the driver was stopped | Delete the bag. Do the test bag first |

### 10.5 Point pair collection

| Symptom | Cause | Remedy |
|---|---|---|
| rviz shows nothing. Fixed Frame shows `No tf data` | The bag is paused (`--start-paused`) | Click the bag terminal. Push the space bar |
| The space bar does nothing | The bag terminal does not have the keyboard focus | Click on the bag terminal first |
| No photo window after a click | The click was not on a point, or Fixed Frame is not `velodyne` | Click on a point of the board. Read the warning in the script terminal |
| `click in 'map', cloud in 'velodyne'` | Wrong Fixed Frame | Set Fixed Frame to `velodyne` |
| `no cloud/image received yet` | Wrong `image_topic`, or the bag is not playing | Use the compressed topic of the bag. Play the bag for 1 s |
| `rms` is large on most pairs | The body of the holder is in the board plane, or the board is not turned | Hold the board at its side. Turn it approximately 45°. Record again |
| `need >=2 rings on the board` | The board is far, small or out of the ring area | Use a nearer position or a larger board |
| You need to go back in the bag | `ros2 bag play` has no rewind key | Start it again with `--start-offset <seconds>` |

### 10.6 Extrinsic result

| Symptom | Cause | Remedy |
|---|---|---|
| Mean error more than 3 px with many `OUT` pairs | Bad pairs (wrong board center or wrong pixel) | Delete the pairs with a large error. Use the photo `posNN.png` to check the pixel |
| A pixel is `1280 720` | That is the image size, not the cursor position (Pinta shows both) | Delete the pair |
| Error is mostly vertical | The 3D points are on a ring, not at the board center | Use `collect_calib_pairs.py`, not manual clicks |
| Error is low but the translation does not agree with the tape measure | Wrong intrinsic values, or the pairs are only in one area | Do Procedure A again with the focus locked. Add positions in all areas |
| `fusion.yaml` lost its comments | `--fusion-out` writes the file again without comments | `git checkout config/fusion.yaml`. Copy only the values |

### 10.7 Build and launch

| Symptom | Cause | Remedy |
|---|---|---|
| `failed to create symbolic link ... existing path cannot be removed` | The workspace was built before without `--symlink-install` | Build without `--symlink-install`, or delete `build/` and `install/` |
| `No known features for CXX compiler "GNU"` | Old CMake cache (for example after a new container) | Delete `build/<package>` and build again |
| `vision.launch.py` starts only `yolo` | The installed launch file is old, or `yolo` comes from a different launch file | `git pull`, build again. Stop the other launch files |
| The topics of the Jetson show on the laptop | Same `ROS_DOMAIN_ID` on the same network | `export ROS_LOCALHOST_ONLY=1` in each laptop terminal |

### 10.8 Files

| Symptom | Cause | Remedy |
|---|---|---|
| The editor stopped and the file is empty | The file was not saved | For nvim: the swap file is in `~/.local/state/nvim/swap/`. Open the file again and select recover |

---

## 11. Reference values

Calibration of November 2026 (C920, 1280 × 720, focus 0, team mount):

| Value | Result |
|---|---|
| fx, fy | 931.46, 939.07 |
| cx, cy | 625.36, 356.76 |
| Distortion (k1, k2, p1, p2, k3) | 0.114, −0.179, −0.0025, −0.0051, 0 |
| Extrinsic pairs | 17, with 15 inliers |
| Mean reprojection error | 1.53 px |
| LiDAR relative to camera | Approximately 11 cm up, 10 cm back, 1 cm to the side |
| Rotation | Approximately 2.4° yaw between the camera and the LiDAR |

**NOTE:** With autofocus on, the same camera gave fx = 912. A focus change
causes a focal length change of approximately 2 % (13 px at the image
edge).
