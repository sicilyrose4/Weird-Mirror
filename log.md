# This is my build log!

9/9/26: I started this project.

<img width="1916" height="1115" alt="sicily writing" src="https://github.com/user-attachments/assets/6cb7d49e-a7cc-448b-b016-49a9a9f5426e" />



# Due 9/16: First build in Touch Designer!

I prompted Claude Code to create a sparkly magic wand effect with my index finger. It used these operators to perform the interaction:

- Used CHOPs to track hand position from a live webcam and drive GPU particle behavior in real time
- Used TOPs to simulate and render a particle system using GLSL shaders with feedback loops

https://github.com/user-attachments/assets/30eaff19-bb54-45b6-9c13-fe776d494be3


# 9/22: Attempted to create a puppet interaction. My prompt and images I provided to Claude shown below.
<img width="2779" height="2425" alt="Puppeteer-02" src="https://github.com/user-attachments/assets/946098cf-5dd7-4f20-aa55-9219840058ad" />
<img width="3680" height="2437" alt="Puppeteer-01" src="https://github.com/user-attachments/assets/e4d0101a-8cad-4655-8237-a0934ab1a8e6" />

## Prompt for Puppeteer Interaction:

You are an expert TouchDesigner developer experienced with real-time computer vision, MediaPipe, and interactive installations. Help me build an interactive piece called "Puppet Strings." Give me a concrete, buildable network: specific operators (TOPs, CHOPs, SOPs, DATs, COMPs), how they connect, key parameter values, and any Python or GLSL I need. Assume I'm on the latest stable TouchDesigner and using the MediaPipe TouchDesigner plugin (Torin Blankensmith's) for hand, pose, and face tracking. If a different tracking approach would work better, say so and explain why.

## Concept
A viewer stands in front of a webcam. At first they see their full body on screen. When they step closer and raise their hands into a "puppeteer" pose, glowing hand outlines appear at the top of the screen showing where to put their hands. When their hands match the outlines, their live body is replaced by a puppet: their own face (cropped live from the webcam) becomes the puppet's head, attached to a stylized puppet body. From then on, their hands work the puppet like a marionette, and moving them moves the puppet's arms and body.

## Hardware / Setup
- Input: a single webcam (1080p, 30fps). Specify resolution and whether to mirror the image.
- Output: one display, landscape, fullscreen via a Window COMP.
- Viewers stand roughly 1.5–3 m away at the start.

## Interaction States (build this as an explicit state machine)
1. IDLE / MIRROR: Show the mirrored live webcam feed with the full body visible. Pose tracking runs in the background.
2. APPROACH: Detect that the viewer has stepped closer. Use shoulder width in normalized pose landmarks, or face bounding-box size, crossing a threshold. Hysteresis prevents flicker at the boundary.
3. GUIDE: Fade in two glowing hand outlines near the top of the screen at fixed target positions (hands raised, palms down, like holding marionette control bars). Also show the viewer's live tracked hands, for example as a softer glow or skeleton, so they can see how close they are. Give feedback as they get closer, such as the outline brightening, pulsing faster, or changing color.
4. LOCK-IN: When both hands stay within a tolerance radius of their targets for a dwell time (about 1 second), trigger the transformation. Include a progress indicator such as a filling ring or brightening outline.
5. PUPPET: Crossfade or animate the live body out and the puppet in. The puppet's head is the viewer's live face, cropped and stabilized using face landmarks with a soft circular or oval mask. Glowing "strings" are drawn from each hand down to the puppet's control points. Hand movement drives the puppet.
6. RESET: If hands are lost for more than about 3 seconds, or the viewer steps back past the distance threshold, return smoothly to IDLE.

Implement the state machine with a clean, debuggable method: a CHOP-based approach (Logic, Timer, Count CHOPs) or a Python state manager in a Text DAT with an Execute DAT. Recommend one and justify it. Expose the current state as a single value I can reference anywhere.

## Puppet Rig and Control Mapping
- Build the puppet as a 2D rig of hinged segments: head, torso, upper and lower arms, and optionally legs, in either SOPs or TOP-based sprite layers. Recommend whichever is easier to art-direct and performs best.
- Mapping:
  - Left and right hand position move the corresponding puppet hand or arm string. Use simple IK or pendulum-style follow so limbs swing naturally.
  - The midpoint and height of both hands controls torso position and bob.
  - The tilt of the line between the hands tilts the torso.
  - Optionally, hand openness (pinch or fist) makes the puppet jump or wave.
- Add smoothing (Filter or Lag CHOP) and a little secondary motion or springiness (Spring CHOP) so the puppet feels like it hangs from strings.
- Keep the face head upright and stable even when the viewer's head moves. Use face landmarks for crop position and scale, and smooth them.

## Visual Style
- Glowing outlines: use a Blur plus Level plus composite bloom, or a GLSL glow. The hand outline shape is still undecided, so offer 2–3 options, for example (a) a stylized hand silhouette, (b) a ring or circle target, (c) a wooden marionette control-bar shape, and show how to swap between them easily.
- Strings: thin glowing lines from the viewer's hands to the puppet joints, with slight sag or sway.
- Puppet body: a stylized look (wooden marionette, paper cutout, or felt). Suggest how to make it swappable with image textures.
- Transitions: every state change should ease in and out rather than cut.

## Performance and Robustness
- Target a steady 60fps output (tracking may run lower). Say where to reduce resolution for tracking vs display.
- Handle missing or flickering detections by holding the last valid value, confidence thresholds, and timeouts.
- Handle left/right hand confusion, especially with a mirrored image.
- Only track the closest or largest person if several people are in frame.
- Include a debug overlay toggle showing landmarks, distance value, match percentage, and the current state.

## Deliverables
1. A network overview diagram described in text, showing major COMPs/containers and data flow.
2. A step-by-step build guide, section by section (Input & Tracking → Distance Detection → State Machine → Hand Guide UI → Match Detection → Face Extraction → Puppet Rig → Strings & Glow → Compositing → Output), with operator names, connections, and key parameter settings.
3. All Python scripts and any GLSL shaders, complete and commented.
4. A list of tunable values (thresholds, tolerances, dwell time, smoothing amounts) gathered in one custom-parameter control panel.
5. Testing and calibration tips for a real installation space.

Ask me clarifying questions first only if something essential is ambiguous; otherwise state your assumptions and proceed.

# 9/23: First pass at interaction for "Plant Yourself" concept

## Prompt:

# Concept
A viewer looks into a computer's webcam and sees themselves on screen, with a strip of dirt along the bottom and a seed floating near the top center. When they reach up and "touch" the seed with their index finger, the seed drifts down and sinks beneath the dirt. Over about ten seconds it grows from a seed into a sprout, then a sapling, then a fully bloomed flower. Just before the flower blooms, the screen tells the viewer to smile and counts down, then takes their photo. When the flower blooms, their face sits in the center of the flower, surrounded by petals, on a full stem with leaves. Then a new seed appears for the next visitor, and over time the dirt fills up with a garden of visitors' face-flowers.

## Setup
- Input: one webcam (use 1280x720 or 1080p at 30fps; recommend one), mirrored so it feels like a mirror.
- Output: one landscape display, fullscreen via a Window COMP.
- Background: the live, mirrored webcam feed, with the dirt strip, seed, flowers, and prompts overlaid on top.
- One viewer at a time. If several people are in frame, use the largest/closest face and that person's hands.

## Sequence and Timing (build this as a clear state machine)
1. WAITING: A seed floats near the top center of the screen, gently bobbing and glowing so it invites a touch. A small glowing dot follows the viewer's index fingertip (MediaPipe hand landmark 8) so they can see what they're touching with. Either hand works.
2. TOUCH: When the index fingertip comes within a set radius of the seed, it counts as touched. Give instant feedback (a small pop, sparkle, or brightness flash). After this, ignore further touches until the next seed appears.
3. FALL (about 1.5 s): The seed drifts down with a slight sway, like it's falling through air, and lands at an open spot along the dirt line (see Garden Layout).
4. PLANT (about 0.5 s): The seed sinks below the top edge of the dirt so it's visibly buried, with a small puff of dirt particles. The dirt strip must render in front of the seed so it's clearly underneath.
5. GROW (about 8 s total, from planting to full bloom):
   - Sprout: a small shoot pokes up through the dirt with two tiny leaves.
   - Sapling: the stem grows taller, more leaves unfurl, and a closed flower bud forms at the top.
   - Bloom: the bud opens and the petals spread around the viewer's face photo.
   Use eased, slightly springy motion so the growth feels organic and continuous rather than jumping between stages.
6. SMILE PROMPT (during the sapling stage, before bloom): Show a clear on-screen prompt such as "Smile!" with a 3-2-1 countdown, timed so the photo is taken about 1–2 seconds before the flower finishes blooming. Add a camera-flash effect when the photo is taken.
7. PHOTO: Capture a single frame and crop the viewer's face (see Face Photo). This is a still image, not live video.
8. BLOOMED: The flower finishes opening with the face photo in its center. Hold briefly, then a new seed fades in at the top center and the cycle returns to WAITING.

The total time from touch to full bloom should be about ten seconds. Put all stage durations in one place so I can tune them.

## Face Photo
- Use face tracking to find the face in the captured frame, and crop tightly around it, expanding the crop upward slightly to include some hair.
- The final image should show only the face and a bit of hair, with no background. Use a circular mask, and if the circle still shows too much background, remove it with MediaPipe selfie segmentation (or a similar matte) before masking. Recommend the cleanest approach.
- If no face is detected at the moment of capture, retry for up to a second or two, then fall back to a friendly placeholder (for example a simple smiley) so the flower still blooms.
- Photos are only shown on screen; nothing is saved to disk.

## Flower Design
- Each flower has a stem, leaves, and a ring of petals surrounding the circular face photo, so together they read as one complete flower.
- Give each flower light variety (petal color, petal count or shape, stem height, slight lean) so the garden doesn't look repetitive.
- Build flowers procedurally (SOPs or GLSL) or from swappable transparent PNG textures; recommend whichever is easier to art-direct, and keep the art swappable so I can drop in my own designs later.
- Bloomed flowers sway gently as if in a breeze.

## Dirt and Garden Layout
- A dirt strip runs across the bottom of the screen (roughly the bottom 12–15%), with a slightly uneven top edge and some texture. It sits in front of seeds and roots and behind stems above the ground line.
- Each visitor's flower stays in the garden. The seed always starts at top center but drifts toward the next open spot along the dirt as it falls, so flowers fill the ground across the screen without overlapping.
- Cap the number of flowers (start around 10–15; suggest a good number for the screen width). When the garden is full, the oldest flower wilts and fades away to free a spot.
- Store each flower's data (position, variation, face photo, age) in a way that scales, such as instancing with a texture array or Replicator COMP. Recommend the approach and explain how each flower keeps its own face photo.

## Robustness and Performance
- Target a steady 60fps output; run tracking at a lower resolution than the display if needed.
- Smooth the fingertip position so the cursor doesn't jitter, and hold the last valid value if tracking briefly drops out.
- Handle left/right hand swapping with a mirrored image.
- If the viewer walks away mid-growth, the flower should still finish growing (with the placeholder face if no photo was taken).
- Include a debug toggle showing hand landmarks, the touch radius, the current state, the stage timer, and the flower count.

## Deliverables
1. A short network overview describing the main COMPs and how data flows between them.
2. A step-by-step build guide: Webcam & Tracking → Fingertip Cursor & Touch Detection → State Machine & Timing → Seed Fall & Planting → Growth Animation → Smile Prompt & Photo Capture → Face Crop & Mask → Flower Assembly → Garden Layout → Compositing → Output, with operator names, connections, and key settings.
3. All Python scripts and any GLSL shaders, complete and commented.
4. A small custom-parameter control panel for the main tunable values (touch radius, stage durations, countdown timing, crop size and hair margin, max flowers, dirt height, sway amount).
5. A few tips for testing and tuning.

Ask me clarifying questions only if something essential is missing; otherwise state your assumptions and proceed.

