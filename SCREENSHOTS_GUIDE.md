# Screenshots Guide — Kirik

Required screenshots for app store submissions.
Take these on a real device or simulator.

---

## Google Play Store

### Required sizes
- Phone: 1080x1920 or 1242x2208 (minimum 2, maximum 8)
- Tablet 7": 1200x1920 (recommended)
- Tablet 10": 1920x1200 (recommended)

### Recommended 8 screenshots (in order):

1. **Splash / Hero** — App icon on the blue background with tagline "Draw. Guess. Win."
   - Simulator: Splash screen on launch
   
2. **Main Menu** — Clean menu with Create/Join buttons
   - Simulate: Launch app after login
   
3. **Game Canvas — Drawing** — Drawer's view with colorful palette, a half-drawn object
   - Simulate: Start game, become drawer, draw something recognizable
   
4. **Game Canvas — Guessing** — Guesser's view with hint dashes and chat bubbles
   - Simulate: As a guesser, show someone typing "is it a... 🐱"
   
5. **Correct Guess** — Green celebration banner "PlayerName guessed it! +350 pts"
   - Simulate: Correct guess moment
   
6. **Leaderboard** — Global leaderboard with medals 🥇🥈🥉
   - Simulate: Navigate to leaderboard
   
7. **Room Lobby** — 6 players ready, code displayed, "START GAME" button
   - Simulate: Room lobby with multiple connected players
   
8. **Profile** — Player profile with stats, level badge, win rate
   - Simulate: Navigate to profile with some dummy stats

---

## Apple App Store

### Required sizes (mandatory for all)
- 6.5" iPhone (1284x2778 or 1242x2688): iPhone 14 Plus / Pro Max
- 5.5" iPhone (1242x2208): iPhone 8 Plus
- 12.9" iPad Pro (2048x2732): if supporting iPad

### Same 8 screenshots as above work for both stores.

---

## How to take screenshots

### Android (Flutter)
```bash
# Connect device or start emulator, then:
flutter run --release
# On device: Volume Down + Power button
# On emulator: Screenshot button in toolbar
```

### iOS (Xcode)
```bash
flutter run --release
# In iOS Simulator: Cmd + S
# On device: Side button + Volume Up
```

### Resize for stores (using ImageMagick)
```bash
# Install: brew install imagemagick
convert screenshot.png -resize 1080x1920^ -gravity center -extent 1080x1920 play_store_phone.png
```

---

## Store Icon (already generated)
- `assets/images/app_icon.png` — 1024x1024 PNG ✅
- Android adaptive icon configured ✅
- iOS AppIcon.appiconset configured ✅

---

## Feature Graphic (Google Play only, 1024x500)
Create a banner image showing:
- Kirik logo on the left
- Screenshot of gameplay on the right
- Blue gradient background (#6C9BD2 → #4A7BB5)
- Tagline: "Draw. Guess. Win."

Use Canva or Figma — free templates available.
