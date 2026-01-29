# VibeLive Design Guide (Standard – Nature Calm)

**Theme / Brand:** Nature Calm (mist sage + olive action)  
**Primary action color:** `#556B2F` (Olive Action)  
**Style:** grounded, low-saturation, long-session friendly  
**Default radius:** `16px`  
**Max content width:** `980px`  
**Font stack:** `ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial`

**Version:** 1.0.0  
**Last Updated (PST):** 2026-01-27 14:30 PST

---

## 1) Core Principles
1. **Mist-first UI:** backgrounds are soft sage/linen; avoid stark white dominance.
2. **Calm contrast:** actions are high-contrast but not neon; text is forest (not black).
3. **One primary action per screen:** only one Olive CTA per view when possible.
4. **Semantic color discipline:** green-ish ≠ success by default; state colors are reserved.

---

## 2) Essential Color Tokens (VibeLive Standard)
Use **only these** unless you have a special reason.

### Surfaces
- `--bg`: `#F1F3F0` (Mist Sage – main background)
- `--card`: `#FAF9F6` (Soft Linen – cards/modals)
- `--soft`: `#E6ECDC` (Olive Soft – subtle highlights/rows)
- `--border`: `#E4E7E3` (Fog Divider)
- `--borderSoft`: `#D7DDD8` (Hover/Emphasis border)

### Text
- `--text`: `#2A302E` (Deep Forest – primary text)
- `--muted`: `#5F6B66` (Ash Green – secondary text)
- `--muted2`: `#9AA5A0` (Muted Moss – helper text)

### Brand Accent (Identity, not CTA)
- `--brand`: `#8DA399` (Dusty Sage – icons/headers/subtle emphasis)
- `--brandSoft`: `rgba(141,163,153,.18)` (soft brand tint)

### Primary Action (CTA)
- `--accent`: `#556B2F` (Olive Action – primary buttons/links)
- `--accentHover`: `#495C28` (hover/pressed)
- `--accentSoft`: `rgba(85,107,47,.12)` (selected background tint)
- `--accentBorder`: `rgba(85,107,47,.30)` (focus/outline)

### Status (Semantic)
**Live / Active (Presence)**
- `--liveBg`: `#E3E9E1`
- `--liveBorder`: `#CBD5C8`
- `--liveText`: `#3A443F`

**Disabled / Offline**
- `--disabledBg`: `#ECEFED`
- `--disabledBorder`: `#E4E7E3`
- `--disabledText`: `#9AA5A0`

**Error / Destructive (Rare)**
- `--dangerBg`: `#F2E8E6`
- `--dangerBorder`: `#E3C7C2`
- `--dangerText`: `#8B4A3E`

---

## 3) Component Rules (Practical + Repeatable)

### Buttons
**Primary (CTA)**
- Background: `--accent`
- Text: `#ffffff`
- Hover: `--accentHover`
- Use for: “Join Room”, “Create Room”, “Start Session”, “Save”, “Confirm”

**Secondary**
- Background: `--card` or `--soft`
- Text: `--text`
- Border: `--border`
- Use for: “Cancel”, “Back”, “Close”

**Danger (Rare)**
- Use `--dangerBg / --dangerBorder / --dangerText`
- Never use Olive for destructive actions

### Links
- Default: `--accent`
- Hover: underline (don’t change color dramatically)

### Inputs
- Background: `--card`
- Border: `--border`
- Focus ring: `--accentBorder`
- Placeholder: `--muted2`
- Error state: use Danger palette

### Cards
- Background: `--card`
- Border: `1px solid --border`
- Hover: border becomes `--borderSoft` OR add a very soft shadow

### Pills / Badges
Keep it **semantic**, not decorative:
- **Live**: `--liveBg / --liveBorder / --liveText`
- **Brand tag**: `--brandSoft` background + `--brand` text
- **Selected row**: `--accentSoft` background + `--accent` text (sparingly)
- **Disabled**: `--disabledBg / --disabledBorder / --disabledText`

---

## 4) Modals, Toasts, Alerts (One Standard Pattern)
**Overlay/backdrop:** `rgba(42, 48, 46, 0.45)`  
**Container:** `--card` with `--border`, radius `16px`

### Alert types
- **Success / Live:** use Live palette
- **Error:** use Danger palette
- **Info:** `--soft` background + `--border` + `--text`

---

## 5) Iconography
- No emojis in UI.
- Use simple stroke icons (Lucide style).
- Sizes: 20 / 24
- Default icon color: `currentColor` (inherits text color)
- Use `--muted` for inactive icons, `--accent` for active icons, `--brand` for decorative identity.

---

## 6) Copy-Paste Tokens (CSS Variables)
```css
:root{
  /* layout */
  --radius: 16px;
  --max: 980px;
  --sans: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;

  /* surfaces */
  --bg: #F1F3F0;
  --card: #FAF9F6;
  --soft: #E6ECDC;
  --border: #E4E7E3;
  --borderSoft: #D7DDD8;

  /* text */
  --text: #2A302E;
  --muted: #5F6B66;
  --muted2: #9AA5A0;

  /* brand */
  --brand: #8DA399;
  --brandSoft: rgba(141,163,153,.18);

  /* actions */
  --accent: #556B2F;
  --accentHover: #495C28;
  --accentSoft: rgba(85,107,47,.12);
  --accentBorder: rgba(85,107,47,.30);

  /* status */
  --liveBg: #E3E9E1;
  --liveBorder: #CBD5C8;
  --liveText: #3A443F;

  --disabledBg: #ECEFED;
  --disabledBorder: #E4E7E3;
  --disabledText: #9AA5A0;

  --dangerBg: #F2E8E6;
  --dangerBorder: #E3C7C2;
  --dangerText: #8B4A3E;

  /* overlay */
  --overlay: rgba(42, 48, 46, 0.45);
}
```

---

## 7) “Do / Don’t” (So VibeLive stays consistent)

**Do**
- Keep the UI misty and low-contrast by default.
- Use Olive primarily for key actions and emphasis.
- Use Dusty Sage for identity (icons/headers), not CTAs.
- Keep error states muted and rare.

**Don’t**
- Add bright greens everywhere (it stops feeling calm).
- Use multiple primary CTAs on the same screen.
- Use pure black text unless required for accessibility edge cases.
- Use emojis as UI icons.
