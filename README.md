# AegisDoor: Smart Residential Entryway System

**Author:** Aayush Keshari  
**Course:** CS 5167 — User Interface Design  
**Project:** Project 1 (Interface to a Smart Object)  

- **Live Application:** (https://aegis-door.vercel.app/)
- **Source Code Repository:** (https://github.com/aayushkeshari/aegis-door)
- **Video Walkthrough (2–3 min):** []

---

## 1. Project Overview & Physical Affordances
AegisDoor is a digital mock-up of an interactive smart residential entry door. Unlike conventional smart locks that condense all interactions onto a single flat lock face, AegisDoor distributes interface controls and displays across **three distinct physical surfaces**:

1. **Exterior Face:** Weatherproof outdoor plane dedicated to greeting and authentication. Features an optical camera lens with a peephole sensor, a backlit 0–9 numeric keypad, an integrated capacitive fingerprint sensor, an NFC tap target, and an illuminated doorbell chime.
2. **Door Edge (Narrow 1.75" Perpendicular Surface):** Houses the mechanical latch mechanism, a magnetic reed frame alignment sensor (verifying Flush vs. Ajar states), a motorized deadbolt throw, and an active multi-segment LED strip providing peripheral feedback on lock and jam status.
3. **Interior Face:** Protected ambient dashboard at eye level providing glanceable security indicators ("(SECURE) All locked"), doorstep package delivery notifications ("3 Packages left delivered on porch [FedEx]"), temporary guest pass configuration, and a physical manual deadbolt slider.

### Norman's Design Principles Applied
- **Affordances:** The flat interior face affords pushing open, while the exterior lever handle affords grasping and pulling.
- **Natural Mapping:** Moving the physical interior slider upward extends the deadbolt into the door frame; sliding it downward retracts the bolt.
- **Signifiers:** Backlit numeric keys signify touch readiness; a pulsing red ring on the camera signifies active optical detection of a visitor.
- **Feedback:** Clear auditory and visual status indicators (color-shifting cards, animated deadbolt status, and edge LED illumination) immediately confirm state changes.

---

## 2. Sensor Assumptions
- **Optical Camera & mmWave Radar:** Senses approaching human visitors and scans the doorstep area for dropped parcels and couriers.
- **Capacitive Touch & Biometric Scanner:** Authenticates registered fingerprints instantly upon handle touch.
- **Magnetic Reed Contact Sensor:** Verifies whether the door is flush within the door frame jamb before motorized deadbolt extension is permitted.
- **Motor Torque / Strain Sensor:** Senses physical resistance or latch obstruction when throwing the deadbolt, immediately reporting mechanical jams.

---

## 3. User Needs & Design Evolution

### User Research Insights
Interviews conducted with three participants identified recurring friction points with existing residential doors:
- **Key & Device Fumbling:** Carrying groceries or packages makes finding keys or using phone apps tedious (addressed by rapid one-touch fingerprint authentication and NFC tapping).
- **Delivery Anxiety:** Uncertainty over package drop-offs while inside the home (addressed by glanceable interior parcel tracking with courier timestamps).
- **Perimeter Assurance:** Nighttime worry about whether the front door is completely latched shut (addressed by high-contrast ambient security status cards visible from across the room).

---

## 4. Implementation Structure & Advanced Levels

Built with **Svelte**, **JavaScript**, and **Vite**, structured around a single reactive state store:

- **Level 0 (Partitioned Viewport):** A strict dual-panel screen layout separating the mock physical door assembly (Device UI) on the left from the interactive Simulation & Testing Harness on the right.
- **Level 1 (Core Object Controls):** Functional PIN keypad with error handling, capacitive Touch ID, dynamic deadbolt animations (`THROWN`, `RETRACTED`, `JAMMED`), and reed sensor alignment.
- **Option 1 (Complex Inputs):** A Guest PIN Access Manager modal to generate schedule-restricted, time-windowed numeric access codes for service workers or guests.
- **Option 2 (Secondary Device):** A mobile companion smartphone frame embedded within the test harness featuring live synchronized push alerts and bidirectional remote locking/unlocking.
- **Option 3 (Usage Data & Profiles):** A dynamic SVG 24-hour hourly access histogram that updates across four distinct usage profiles (*Working Professional*, *Busy Family*, *Airbnb Host*, and *Vacation / Away*).
- **Option 4 (24-Hour Time Simulation):** An accelerated day-cycle engine (1 second = 1 hour) that simulates daily routines, morning departures, parcel deliveries, and automatic nighttime lockdowns.

---

## 5. Future Work
- **Motorized Automatic Swing:** Integrating automated door operators for hands-free and wheelchair-accessible entry.
- **Ultra-Low-Power E-Ink Surfaces:** Exploring reflective e-paper technology for the interior display to maximize battery endurance during power outages.
- **Matter / Thread Protocol Integration:** Unifying the door's edge sensor network with broader home automation systems.

---

## 6. AI Documentation
AI assistance (Gemini) was utilized as an interactive design and coding thought partner throughout this project. It helped ideate the multi-surface interface layout, verify Norman's design affordance mappings, write the Svelte prototype code and simulation harness, and resolve cross-platform build configuration errors between macOS ARM64 and Linux deployment environments.
