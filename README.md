# Twist 5: Pedestrian Speed and Clearance Time

Track pedestrians on a crossing, estimate each person's crossing speed, and work out how long the crossing needs to clear.

**Input:** a video of the crossing.
**Output:** the same video with a box on every person, their **ID** and their **speed (m/s)**, plus a table of speeds and the **clearance time**.

> Course: C389 | Team: **3** | Members: **ASMITA SIVACHANDRAN,CAI XINGMENG, GE YOUYOU, HTIN AUNG KYAW, RAINIE NG YI YUAN, RANJITHRAJ RAJAVEL**

---

## How it works

1. **Detect.** A YOLO model fine-tuned on our own crossing photos finds each person and gives a direction class, `person_left` or `person_right`. This is the baseline model from Lesson 3.
2. **Track.** ByteTrack links the detections over time so each person keeps one ID.
3. **Cross the marks.** Two lines (**mark A** and **mark B**) are drawn across the crossing. For each person we find the exact moment their **feet** cross each mark, interpolated between frames.
4. **Speed.** `speed = D / (time at B - time at A)`, where **D** is the distance between the two marks, measured with a tape on site. No pixel-to-metre conversion is needed.
5. **Clearance time.** `clearance time = W / slowest walking speed`, where **W** is the kerb-to-kerb width. We use the slowest walker, not the average.
6. **Draw.** The marks, boxes, IDs and speeds are written onto the output video.

Everything is in one notebook: [`T5_speed_demo.ipynb`](T5_speed_demo.ipynb). It combines the deployment ideas from Lesson 4 (load the model once, refuse bad input clearly, one function that does the whole job, a Gradio page) with the tracking and marking labs from Lesson 5.

---

## Repository layout

```
.
├── README.md
├── T5_speed_demo.ipynb        # the whole pipeline, with an explanation above every code cell
└── report/                    # [optional] report, screenshots, demo video link
```

Large files are **not** committed. On Google Drive the notebook expects:

```
MyDrive/C389/
├── artifacts/L3/best.pt       # trained model weights from Lesson 3 (or the fine-tuned version)
├── data/clips/walkway.mp4     # the crossing video
└── artifacts/T5/              # created by the notebook: output video, speeds.csv, results.json
```

---

## Running it (Google Colab)

1. Upload your clip to Drive as `data/clips/walkway.mp4`, and make sure `artifacts/L3/best.pt` is there.
2. Open `T5_speed_demo.ipynb` in Colab and set the runtime to **T4 GPU**.
3. Run the cells **one at a time, in order**.
   - **Section 2:** check the camera did not move, then **click four points** on the first frame: two on mark A, then two on mark B. Put the two points of each mark as far apart as you can.
   - **Section 3 (settings):** the only cell you edit. Set `MARKS_METRES` (D) and `WALKWAY_LENGTH_M` (W) **in metres**, and `MODEL_PATH` if you have a fine-tuned model. Run it again after clicking so it picks up your points.
   - **Section 3 (preview):** confirm the marks sit on the real lines on the first, middle and last frames.
   - Then run the rest. The demo video is saved as `artifacts/T5/t5_demo_output.mp4`.

Without `best.pt` the notebook falls back to the stock YOLO model. It still finds people and measures speed, but cannot tell left from right.

### Settings that matter


| Setting                        | Meaning                                                                                   |
| ------------------------------ | ----------------------------------------------------------------------------------------- |
| `MARKS`                        | Mark A and mark B, two points each. Left as`None` to use the points clicked in Section 2. |
| `MARKS_METRES`                 | **D**: tape distance between mark A and mark B, in metres.                                |
| `WALKWAY_LENGTH_M`             | **W**: kerb-to-kerb length, in metres. `None` means use D.                                |
| `MIN_WALKING_SPEED`            | Slower than this (m/s) counts as standing and is left out of the clearance time.          |
| `TRACK_BUFFER`, `MATCH_THRESH` | ByteTrack settings. Section 15 shows where each one breaks on our clip.                   |

---

## Site measurements for our clip


| Item                                   | Value           |
| -------------------------------------- | --------------- |
| Distance between mark A and mark B (D) | **[FILL IN] m** |
| Kerb-to-kerb width (W)                 | **[FILL IN] m** |
| Date recorded                          | **[FILL IN]**   |
| Camera position                        | **[FILL IN]**   |

## Results


| Measure               | Value             |
| --------------------- | ----------------- |
| People measured       | **[FILL IN]**     |
| Average walking speed | **[FILL IN] m/s** |
| Slowest walking speed | **[FILL IN] m/s** |
| Clearance time        | **[FILL IN] s**   |

Demo video: **[FILL IN link]**

---

## Capturing a clip that works

None of this can be fixed afterwards.

- **The phone must not move.** Use a tripod, clamp or wall. The notebook checks the first and last frames and warns if the camera drifted.
- **Make the marks visible.** Put tape or chalk lines across the ground before filming, then measure between those lines with a tape.
- **Start recording before the person enters, and keep going until they are well past mark B.** A person already past mark A in the first frame, or still short of mark B in the last, cannot be measured.
- **One person at a time** through the marks. People who overlap can swap IDs.
- **30 fps or more**, landscape, side-on or a slight angle, with both marks and the whole crossing in view.
- Include a **slow walker**, because the clearance time depends on the slowest speed. If walks are staged, say so in the report.

## Known limitations

- A person who walks at an angle covers more ground than D, so their speed reads low. Side-on filming keeps this small.
- A speed is the average between the two marks, not the speed at any single moment.
- Someone the tracker splits into two IDs, or who crosses only one mark, is listed as **excluded** with a reason. The notebook never invents a speed for them.
- The clearance time is only as good as the slowest person who happened to be in the clip.
- Marks belong to one camera position. A clip from a different position needs new marks.

---

## Requirements

- Python 3, Google Colab (the click-to-mark tool works on Colab only)
- `ultralytics==8.4.120`, `lap`, `gradio` (installed by the notebook's first cell)
- OpenCV, Pillow, PyYAML (preinstalled on Colab)

---

## Acknowledgements

Built on [Ultralytics YOLO](https://github.com/ultralytics/ultralytics) and its ByteTrack tracker. Course material from Lessons 3, 4 and 5 (C389).
