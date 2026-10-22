# 481 - Design Log Team #46

A chronological record of the team's design research, meetings, and decisions.

> **How to read this log:** Each entry lists its type, an objective, the findings or discussion, and (where applicable) open questions and action items. Every action item has an **Owner**, listed by first name.

---

## September 17, 2026: Swipe-to-Type Software Approach

**Type:** Research
**Author:** Nada Salem
**Objective:** Identify a software approach for swipe-to-type and define the supporting development tooling.

### Approach
Swipe-to-type is the main software problem for this project. Developing it from scratch would be too expensive, so the team will build on open-source implementations instead. Both classical and neural implementations exist, and the pros and cons of each will be weighed.

| Implementation | Type | Approach | Notes |
|---|---|---|---|
| [CleverKeys](https://github.com/tribixbite/CleverKeys) | Neural | ONNX-based neural model to decipher gestures | — |
| [FlorisBoard](https://github.com/florisboard/florisboard) | Classical | Its own statistical gesture classifier | Popular, but still in beta |
| [AnySoftKeyboard](https://github.com/AnySoftKeyboard/AnySoftKeyboard) | Classical | Geometric path-matching algorithm | — |

### Required Adaptations
- Most candidates are Android keyboard apps. Only the gesture-decoding algorithm is needed, so the rest of each app must be stripped away.
- Licensing must be checked: open source does not mean unrestricted. GPL-3.0, for example, has a copyleft requirement, while Apache-2.0 does not.
- The algorithms are complex and numerous, which makes selection difficult and costly to get wrong. An approach should be chosen only once the underlying algorithm is well understood, reusing existing work wherever possible.

### Debugging GUI
- A GUI for testing and debugging is needed as development progresses, and a good process should be established early.
- Proposed: a simple Python application built with PyQt. Nada has prior Qt experience.
- Requirements are still to be defined. Visual representation of data will help, especially for firmware tasks that may be disjointed from software development.

### Symposium Website
Low priority, but recorded here so it is not forgotten as a software task.

### Action Items

| # | Action | Owner | Status |
|---|---|---|---|
| 9/17-1 | Understand the underlying swipe-to-type algorithms, then choose an approach | Nada | Closed |
| 9/17-2 | Review licensing (e.g. GPL-3.0 copyleft vs. Apache-2.0) for each candidate repo | Nada | Closed |
| 9/17-3 | Strip the gesture-decoding algorithm out of the Android keyboard apps | Nada | Closed |
| 9/17-4 | Define requirements for the debugging setup | Nada | Open |
| 9/17-5 | Build a simple PyQt debugging GUI | Nada | Closed |
| 9/17-7 | Symposium website (low priority, later) | Nada | Open |

---

## September 17, 2026: Sensing Architecture Options

**Type:** Research
**Author:** Ryan Scomazzon
**Objective:** Compare candidate sensing architectures for the ring and wristband form factors, and survey other form factors.

### Ring

| Architecture | Pros | Cons | Risk level | Links |
|---|---|---|---|---|
| Optical flow + time-of-flight (TOF) + IMU | • Redundant data points for data fusion | • Does not detect small movements of the distal finger joint<br>• Off-the-shelf optical flow sensors are not designed for close range, so a custom lens is required<br>• Difficult to detect when the finger is raised from the surface<br>• Only works on surfaces with a lot of detail | TBD | — |
| IMU + flex detection | • Accurate representation of distal movement | • High drift expected when using only the IMU for yaw<br>• No muscles are in the finger, so ECG cannot be used for flex<br>• A force-sensor approach would have a lot of noise from other movements | TBD | — |
| Thumb ring camera | • Absolute reference point minimizes drift<br>• Versatile | • Computer vision (CV) requires external compute and introduces latency from bandwidth limitations | TBD | — |

### Wristband

| Architecture | Pros | Cons | Risk level | Links |
|---|---|---|---|---|
| Ultrasonic/TOF triangulation + IMU for tap detection | • Self-contained<br>• Likely accurate position | • Signal may reflect off the skin or detect too high a point on the finger<br>• Cannot detect release in the same spot as a tap | TBD | — |
| Mouse sensor on skin to detect deformation directly under the wristband | • Self-contained<br>• Can detect finger release if the finger has moved | • Likely difficult to differentiate strong close and near-far movements (possibly acceptable)<br>• Cannot detect release in the same spot as a tap | TBD | — |
| AC phase shift detection | • Very accurate | • Needs a ring<br>• May be difficult to implement | TBD | [Demo video](https://www.youtube.com/watch?v=9hu8MNuvCHE) |
| Camera facing the wrist | • Self-contained<br>• High accuracy<br>• Versatile | • Excess compute needed for CV<br>• Bandwidth limitations introduce latency for offboard compute | TBD | — |
| Neural band detecting pitch and yaw of the index finger with ECG | • As intuitive as a ring | • Less direct sensors, so reading the data may be difficult | TBD | — |

### AC Phase Shift: Reference Paper
The approach is based on [SkinTrack](https://modernmobile.cs.washington.edu/docs/skintrack.pdf).
- The authors achieved an average error of about 7 mm. Even reaching within 50% of that (about 10 mm error) would not be good enough for our purposes.
- They used the AD8302 phase comparator.
- Issues they noted:
  - Accuracy changes throughout the day.
  - Pushing harder registers as motion.
  - An ML model was used to get positions, though fairly reliable classical relations were also found.
- Phase shift finger detection experiments are planned; results will be documented.

### Other Form Factors
- **Tablet in pocket:** interaction through fabric.
- **Inside-out glasses tracking:** finger contact detection would be very difficult, and line-of-sight issues are likely.
- **Gaze-based eye tracking:** not discreet; users would look very odd.

### Open Questions
- Can skin deformation, sensed optically, be used for pitch detection?

---

## September 20, 2026: IMU Research

**Type:** Research
**Author:** Akshaya
**Objective:** Shortlist candidate IMUs for the ring.

### Candidate Parts

| Part | Axes | Key features | Price | Link |
|---|---|---|---|---|
| Bosch BMI270 | 6-axis (accelerometer + gyroscope, capturing acceleration and rotation) | • Small<br>• Power efficient | $6.61 | [DigiKey](https://www.digikey.ca/en/products/detail/bosch-sensortec/BMI270/9974486) |
| STMicroelectronics LSM6DSV16X | 6-axis (accelerometer + gyroscope) | • Embedded machine-learning core runs motion recognition and event detection inside the IMU, cutting host processing and power<br>• Basic tap or motion detection runs on the chip without waking the main microcontroller | $8.98 | [DigiKey](https://www.digikey.ca/en/product-highlight/s/stmicroelectronics/lsm6dsv16x-inemo-3-axis-accelerometer-and-gyroscope) |

### Open Questions
- Is on-chip tap and motion detection (without waking the main microcontroller) something the design needs?

---

## September 22, 2026: Weekly Meeting

**Type:** Meeting
**Objective:** Settle IP handling and housekeeping, and narrow down how to detect touch and fingertip position for the ring.

### IP
- Groups have patented after the symposium before. However, a project that is publicly disclosed before a patent is filed creates a legal loophole risk.

### Housekeeping
- A separate private GitHub repo will hold private code.
- WARG is running a sensor fusion event series on September 22 at 8 pm (E7 5343).

### Touch Detection
- The IMU can pick up vibrations, but it is unclear how to detect the finger leaving the skin surface.
- Candidate ways to determine whether a touch registered:
  - Wristband-to-ring AC implementation
  - A software check for whether a word is completed
  - Muscle actuation and blood flow changes in the finger during touch
- Optical flow sensors are mostly designed for 8 cm or more, so the lens would have to be replaced.

### Fingertip Sensing Modes
- Infrared to detect the fingertip.
- Ultrasonic to detect the fingertip.

### Software
- Look into what bigger companies are using.
- Implement on a trackpad as a proof of concept.

### Action Items

| # | Action | Owner | Status |
|---|---|---|---|
| 9/22-1 | Set up a separate private GitHub repo for private code | Nada | Closed |
| 9/22-2 | Investigate muscle actuation and blood flow changes during touch | Ben | Closed |
| 9/22-3 | Look into what bigger companies are using for software | Unassigned | Open |
| 9/22-4 | Implement on a trackpad as a proof of concept | Nada | Closed |
| 9/22-5 | Think of DOEs to test what is viable for the ring | Ryan | Closed |
| 9/22-6 | Trial the FIGlab implementation | Ryan | Open |
| 9/22-7 | Test with a drawing tablet | Shruthi | Closed |

---

## September 24, 2026: Finger Biomechanics Research

**Type:** Research
**Author:** Ben Ngo
**Objective:** Evaluate whether finger anatomy supports blood-flow-based tap detection, and identify alternative ways to track contact.

### Hand Anatomy
Each finger has distal, middle, and proximal phalanges.

### Blood Flow Tracking for Finger Tap
Blood flow tracking is unlikely to work, for two reasons:
1. Blood flow changes are mostly contained in the distal capillaries, so a change in the proximal phalanx is unlikely.
2. People will likely tap with the tip of the finger, which means contact at the nail as well.

### Alternative Contact Tracking
- Capacitive touch on the side of the ring for a "writing mode".
- Capacitive touch on the side of the ring (similar to wireless earbud controls) could also provide backspace, undo, and other commands.
- The input may not need to be tapping a surface at all. Touching the index finger and thumb together could serve as the input.

---

## September 24, 2026: Tracking Finger Position in 3D Space

**Type:** Research
**Author:** Ben Ngo
**Objective:** Outline how to track the general position of the fingertip in 3D space.

### Approach
- An IMU provides the general position tracking.
- Candidate methods for tracking the fingertip relative to the ring:
  - Optical flow
  - TOF
  - Ultrasonic

### Open Questions
- How should fingertip position be mapped to an algorithm that detects handwriting?
- How should the system calibrate to a surface used as a keyboard, especially one that is not flat (e.g. the hand)?

### Timeline
- 2D tracking of the fingertip in a larger form factor by EOT.

---

## September 29, 2026: Weekly Meeting

**Type:** Meeting
**Objective:** Prepare the requirements presentation, compare battery options, and review finger tracking and IMU choices.

### Presentation
- Open question: is there a minimum displacement criterion?
- Criteria and constraints need to be separated and cleaned up.
- **Criteria:**
  - Accuracy vs. precision on the keyboard
  - Resolution, tuned down to 0.5 mm, with the usable range cleaned up
- The presentation is on September 30 at 4 pm.

### Software
- Test the feel of a drawing tablet for swipe-to-type.

### Battery
- Candidate: a 3.7 V, 20 mAh special-shaped cell (UFine UFX0947-13; datasheet on ufinebattery.com).
- Other suppliers to consider: LiPol (smart ring LiPo), UFine (curved batteries), lipobattery.us.
- Grepow can go down to a minimum order quantity (MOQ) of 5, but shipping is expensive.

### Finger Tracking
The preferred stack will be chosen at the next weekly meeting.

| Approach | Notes |
|---|---|
| Strain sensors on the phalanges | Sideways motion could also trigger the strain sensor. Possibly manageable with sensor fusion plus a pressure sensor. |
| Force myography (FMG) and pressure sensing | To investigate |
| Optical sensor tracking the skin surface | A mouse is needed for prototyping, but there is no compatible dongle |
| Acoustics | Reference: Ring-a-Pose (ACM, 10.1145/3699741) |
| Light sensor to the fingertip | To investigate |

### IMU
- Two candidate parts. Size and power efficiency should be about the same for both.
- The STM part has embedded AI, with basic tap and motion detection and state machine logic on the chip.
- A 9-axis IMU lets the magnetometer correct for drift.
- Open questions: what sampling rate is targeted, and what is the power efficiency?

### Action Items

| # | Action | Owner | Status |
|---|---|---|---|
| 9/29-1 | Test the feel of a drawing tablet for swipe-to-type | Nada | Closed |
| 9/29-2 | Clean up criteria vs. constraints | All | Closed |
| 9/29-4 | Add a slide on the main approaches | Ben | Closed |
| 9/29-5 | Revisit finger tracking and choose a preferred stack | All | Open |
| 9/29-6 | Find out what the STM embedded AI does and whether it saves compute power | All | Open |
| 9/29-7 | Keep investigating the IMU: power efficiency, sampling rate, 9-axis | All | Open |

---

## October 8, 2026: Weekly Meeting

**Type:** Meeting
**Objective:** Rework the requirements and assign sections of the PDP.

### Terminology
- **Objective:** what is ideal (want).
- **Constraint:** whether a solution is viable (need).
- **Criteria:** which option is better (compare, via the decision matrix).

### Open Questions on Requirements
- Do constraints have to be quantifiable? Is safety a constraint?
- Can a parameter be an objective, with a stricter value of the same parameter as a constraint? For example, a 0.5 cm resolution objective vs. a 1 cm resolution constraint. An example should be added to clarify.
- Should the PDP restate the motivation and background? Probably yes.
- Why not put all objectives in the criteria? So far only the most important ones were included.
- How should requirements be defined across form factors?
  - A high-level decision matrix for the general solution (ring vs. watch vs. eye tracking) looks very different from one for the technology within a solution (sensing types).
  - Precision and accuracy depend heavily on the solution and input scheme chosen.
  - Are we expected to compare general solutions or implementations, and which is requested for the ring?

### IMU Progress
- An Arduino and IMU were picked up. Lockers are available in E2 (rent from EngSoc, $10 deposit).
- The IMU cannot distinguish a moving finger from a rotating body.
- Option: use an IMU with an onboard processor that performs sensor fusion and noise filtering. Its power consumption is still being checked.

### Action Items

| # | Action | Owner | Status |
|---|---|---|---|
| 10/8-1 | Write PDP: Introduction & Need Identification (incorporate feedback) | Ryan | Closed |
| 10/8-2 | Write PDP: Problem Definition, Objectives, Constraints, and Criteria | Ryan | Closed |
| 10/8-3 | Write PDP: Alternate Design Solutions and Design Selection Matrix | All | Closed |
| 10/8-4 | Write PDP: Patent Review | Nada, Shruthi, Ryan | Closed |
| 10/8-5 | Write PDP: Project Timeline | Ben | Closed |
| 10/8-6 | Prepare PDP: Quality of Presentation | All | Closed |
| 10/8-7 | Check the power consumption of the IMU with an onboard processor | Akshaya, Shruthi | Closed |

## October 14, 2026: Weekly Meeting

**Type:** Meeting  
**Objective:** Review sensor and battery procurement, clarify PDP requirements and design selection expectations, and finalize presentation responsibilities.

### Sensor Research and Procurement

An Instagram creator's finger-tracking project was identified as a useful reference for the team to review. She initially experimented with an IMU but appears to have abandoned that approach. Reverse-searching her resources led to an optical flow sensor.

Current sensor procurement plans:

- **Optical flow:** Purchase a sensor from AliExpress.
- **TOF:** Borrow a time-of-flight sensor from MES rather than purchasing one. The sensor was already signed out, so availability needs to be checked in person. Infrared and ultrasonic alternatives are also available.
- **IMU:** Continue using the IMU already obtained from MES. Although its chip is end-of-life (EOL), purchasing a replacement is not yet necessary.

Additional sensor purchases can be deferred while the team evaluates the components available through MES.

### Battery Research

- **AliExpress:** The closest suitable battery found has a capacity of 15 mAh.
- **Specialty supplier:** Minimum order quantity (MOQ) is 5 units, with a fixed shipping cost.
- **Group purchasing:** Mahir's group may also be interested in small batteries, potentially allowing the teams to share shipping costs.

### PDP Requirements Clarifications

**Objectives, Constraints, and Criteria**

- Constraints do not all need to be quantifiable. Qualitative constraints, such as ergonomics and material safety, should still be included.
- Avoid defining the same parameter as both an objective and a constraint using different thresholds (e.g., 1-hour battery life as an objective and 5-hour battery life as a constraint).
- Such parameters should instead be placed under constraints or used as evaluation criteria.
- Objectives may also be included as criteria in the design selection matrix.

**Design Selection Matrix**

- At least three alternative design solutions must be considered in the PDP.
- The comparison can remain at a high level; detailed technical decisions about individual sensors are not required.
- A high-level comparison may limit the ability to evaluate technical requirements, since different form factors (e.g., ring vs. wristband) may have different requirements.
- Evaluation criteria should be measurable where possible. Examples include:
  - Ease of use, approximated by the number of components.
  - Bill of materials (BOM) cost.

**Presentation**

- Slides from previous presentations may be reused.
- The Introduction and Need Identification section can build on previous work, incorporating feedback and reducing unnecessary detail.
- Strong evidence supporting the need for the project should be retained.

### PDP Responsibilities

| Section | Weight | Owner |
|---|---|---|
| Introduction & Need Identification | 20% | Ryan |
| Problem Definition, Objectives, Constraints, and Criteria | 20% | Akshaya |
| Alternate Design Solutions and Design Selection Matrix | 20% | Ben, Akshaya |
| Patent Review | 20% | Nada, Shruthi, Ryan |
| Project Timeline | 5% | Ben |
| Quality of Presentation | 15% | All |

### Action Items

| # | Action | Owner | Status |
|---|---|---|---|
| 10/14-1 | Review the Instagram creator's finger-tracking project and resources | All | Closed |
| 10/14-2 | Purchase an optical flow sensor from AliExpress | Ryan | Closed |
| 10/14-3 | Visit MES to check TOF sensor availability and investigate infrared/ultrasonic alternatives | Shruthi | Open |
| 10/14-4 | Prepare PDP: Introduction & Need Identification, incorporating previous feedback | Ryan | Closed |
| 10/14-5 | Prepare PDP: Problem Definition, Objectives, Constraints, and Criteria | Akshaya | Closed |
| 10/14-6 | Prepare PDP: Alternate Design Solutions and Design Selection Matrix | Ben, Akshaya | Closed |
| 10/14-7 | Prepare PDP: Patent Review | Nada, Shruthi, Ryan | Closed |
| 10/14-8 | Prepare PDP: Project Timeline | Ben | Closed |
| 10/14-9 | Finalize PDP presentation slides and review presentation quality | All | Closed |

## October 21, 2026: Weekly Meeting
**Type:** Meeting  
**Objective:** Prepare for PDP presentation.

Not much to report here, besides rehersal for PDP on the 22nd. Await feedback from instructors and peers to move forward.
