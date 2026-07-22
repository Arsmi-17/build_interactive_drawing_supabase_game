# Interactive Drawing

A Unity 2D drawing toy set underwater. You draw inside a circular pad, submit it, and your
drawing *comes alive* — it swims out into the ocean scene as a physics object that drifts,
bounces off the other drawings, and tells you something nice about what you made. Everything
anyone draws goes into a shared library you can browse, like, favourite and download.

Built for the web (Unity WebGL) and published on the Arsmi Games platform.

---

## How you play

1. **Tap the toggle** to open the circular drawing pad. It slides up from the bottom, pulsing
   until you notice it.
2. **Draw.** Pick one of six colours from the ring around the pad. Two tools:
   - **Brush** — paint strokes, freehand
   - **Paint bucket** — flood-fill an enclosed area
   - **Undo** steps back through your last 20 states; **Reset** clears the pad.
3. **Submit.** The pad clears itself, ready for the next one.
4. **Watch it come alive.** Your drawing surfaces from behind the pad as a floating object with
   real 2D physics — noAC gravity, constant drift, bouncing off other drawings and off the edges
   of the screen.
5. **Get a compliment.** A moment after it starts floating, the drawing pauses and a speech
   bubble appears with a message picked to fit *what you actually drew* — see below.
6. **Tap a floating drawing** to select it. From there you can email it to yourself, or remove
   it from the scene.
7. **Open the library** to see every drawing — yours, everyone's, and the ones you favourited.

There is no score, no fail state, and no timer. It is a sandbox.

---

## Features

### The drawing pad
- Circular canvas with live per-pixel texture painting (512×512), clipped to the circle.
- Brush and paint-bucket tools, six-colour palette laid out in a ring around the pad.
- 20-step undo history, plus reset.
- Animated show/hide, with a pulsing toggle button so the pad is always findable.

### Drawings that come alive
- Each submitted drawing becomes a runtime `SpriteRenderer` object with a `Rigidbody2D`,
  `CircleCollider2D` and a bouncy `PhysicsMaterial2D`.
- Gravity-free constant drift, frozen rotation, elastic collisions between drawings.
- Clamped inside the camera view, so nothing ever wanders off-screen.
- A cap on how many can float at once, so the scene stays readable.
- Tap to select — the drawing pauses and reveals its actions.

### Compliments that actually read your drawing
When you submit, the texture is analysed — how many pixels you filled, how many distinct colour
buckets you used, how much of the pad you covered, how it's laid out — and the message is chosen
from the matching set: tiny drawings, single-colour drawings, colourful ones, full-pad ones, or
ones with interesting composition. Draw one small black dot and you get a different response than
if you fill the pad with six colours.

### The shared library
- Three tabs: **Global** (everyone's drawings), **Mine**, and **Favourites**.
- Responsive grid — three columns in landscape, two in portrait.
- **Like / dislike**, with your own reaction remembered per drawing.
- **Favourite** to keep a drawing in your own tab.
- **Download / email** any drawing to yourself.
- **Delete** your own drawings.

### Profiles
- A username and an avatar you pick from a set, editable from the profile popup.
- Signed-in players get their name from their Arsmi Games account; drawings are keyed to a
  per-game pseudonymous player id, never to anything you type.
- Guests can draw and play freely — they're prompted to sign in only when they try to save
  something that should outlive the session.

### Sharing by email
- Tap the download icon on a floating drawing or a library item and a reusable email popup
  opens above it (the drawing pauses while it's open; click outside to dismiss).
- The PNG is uploaded to Supabase Storage, and a styled HTML email arrives with the drawing
  embedded and a link to open it full size.
- Desktop and Editor builds send over SMTP directly; WebGL can't open an SMTP socket from the
  browser, so it posts to a small Node endpoint that holds the credentials server-side
  ([Backend/](Backend/)).

### Presentation
- Procedural ocean scene: depth-graded background, ~80 drifting bubbles and ~36 sparkles,
  generated at runtime rather than authored as assets.
- Layout re-reads the screen every frame, so it fits phones, tablets and desktop — portrait and
  landscape — without separate scenes.

---

## Built with

Unity 2D (URP) · C# · Unity UI + TextMeshPro · Unity 2D physics · UnityWebRequest ·
Supabase (Storage + PostgREST) · Node/SMTP email backend · Next.js admin dashboard ·
Arsmi Games GameHub SDK

## How the code is organised

Each script owns exactly one job, and they talk through events rather than reaching into each
other:

| | |
|---|---|
| [CircleDrawingPad.cs](Assets/Scripts/CircleDrawingPad.cs) | the canvas — pixels, tools, undo |
| [CirclePadControls.cs](Assets/Scripts/CirclePadControls.cs) | the buttons — raises events, draws nothing |
| [CircleDrawingAppController.cs](Assets/Scripts/CircleDrawingAppController.cs) | listens to those events, runs the flow, picks the compliment |
| [FloatingDrawingManager.cs](Assets/Scripts/FloatingDrawingManager.cs) / [FloatingDrawing.cs](Assets/Scripts/FloatingDrawing.cs) | spawning, physics, selection, the message bubble |
| [DrawingLibraryController.cs](Assets/Scripts/DrawingLibraryController.cs) / [LibraryDrawingItemView.cs](Assets/Scripts/LibraryDrawingItemView.cs) | the library grid and its tabs |
| [UserProfileManager.cs](Assets/Scripts/UserProfileManager.cs) / [UserProfilePopupController.cs](Assets/Scripts/UserProfilePopupController.cs) | identity, name, avatar |
| [SupabaseStorageUploader.cs](Assets/Scripts/SupabaseStorageUploader.cs) / [SupabaseInteractiveDrawingDataStore.cs](Assets/Scripts/SupabaseInteractiveDrawingDataStore.cs) | file upload; rows, likes and favourites |
| [SmtpEmailSender.cs](Assets/Scripts/SmtpEmailSender.cs) / [EmailSendBox.cs](Assets/Scripts/EmailSendBox.cs) | sending; the popup that asks for the address |
| [OceanThemeEffects.cs](Assets/Scripts/OceanThemeEffects.cs) / [ResponsiveUiController.cs](Assets/Scripts/ResponsiveUiController.cs) | the ocean, and fitting any screen |
| [GameHubPlatformIntegration.cs](Assets/Scripts/GameHubPlatformIntegration.cs) | the platform: handshake, mute, fullscreen, identity |

## Running it

Open the project in Unity and play
[Assets/Scenes/Interactive Drawing.unity](Assets/Scenes/Interactive%20Drawing.unity). Cloud
features need a Supabase project — apply [Supabase/](Supabase/) and fill in the URL and anon key
per [.env.example](.env.example). Without them the drawing and floating half still works; the
library and email won't.

## More reading

- [PUBLISHING.md](PUBLISHING.md) — building for WebGL and uploading to Arsmi Games
- [project-architecture.md](project-architecture.md) — how the pieces fit together
- [project-explanation.md](project-explanation.md) — a walkthrough of each flow
- [Backend/README.md](Backend/README.md) · [admin/README.md](admin/README.md)
