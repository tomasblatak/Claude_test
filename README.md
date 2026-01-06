# 🎨 Animated Logo & Marketing Automation

This repository contains:
- **Animated SVG logos** - Different animation styles perfect for email profile pictures
- **N8N Workflow** - Google Sheets to Leadspeaker.com automation for contact management

---

## 🎨 Animated Logo for Email Profile Picture

Animated SVG versions of your logo with different animation styles, perfect for use as an eye-catching email profile picture.

## 🎬 Animation Styles

### 1. **Pop & Pulse** (`logo.svg`)
- Smooth pop-in entrance
- Gentle continuous pulse
- Professional yet engaging
- **Best for:** Business emails, professional communication

### 2. **Bounce & Float** (`logo-bounce.svg`)
- Energetic bounce entrance
- Floating motion with glow effect
- Very attention-grabbing
- **Best for:** Marketing emails, creative industries

### 3. **Spin & Rotate** (`logo-rotate.svg`)
- Spinning entrance effect
- Subtle rotation movement
- Playful and memorable
- **Best for:** Tech startups, casual communication

### 4. **Zoom & Breathe** (`logo-zoom.svg`)
- Powerful zoom-in effect
- Breathing motion
- Bold and confident
- **Best for:** Sales, promotional emails

## 🚀 Quick Start

1. **Preview the animations:**
   - Open `preview-all.html` in your browser to see all styles side-by-side
   - Or open `animated-logo.html` for a single animation view

2. **Choose your favorite style**

3. **Convert to GIF** (see instructions below)

## 📦 Converting SVG to GIF

### Method 1: Online Tool (Easiest) ⭐
1. Go to [ezgif.com/svg-to-gif](https://ezgif.com/svg-to-gif)
2. Upload your chosen SVG file
3. Settings:
   - Width: 200px
   - Height: 200px
   - Duration: 3-4 seconds
4. Click "Convert to GIF"
5. Download and optimize if needed

### Method 2: Screen Recording
1. Open the HTML preview in your browser
2. Use a screen recorder:
   - **Mac:** QuickTime Player → File → New Screen Recording
   - **Windows:** Windows Game Bar (Win + G)
   - **Linux:** OBS Studio or SimpleScreenRecorder
3. Record 3-4 seconds of the animation
4. Convert video to GIF at [ezgif.com/video-to-gif](https://ezgif.com/video-to-gif)
5. Crop to square and resize to 200x200px

### Method 3: Using FFmpeg (Advanced)
```bash
# After recording the animation
ffmpeg -i recording.mp4 -vf "fps=10,scale=200:-1:flags=lanczos" -loop 0 output.gif

# Optimize the GIF size
ffmpeg -i output.gif -vf "fps=10,scale=200:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 optimized.gif
```

## 💡 Tips for Email Profile Pictures

- **File Size:** Keep under 200KB for best email client compatibility
- **Dimensions:** 200x200px or 150x150px recommended
- **Duration:** 2-4 seconds is optimal
- **Format:** GIF is most compatible across email clients
- **Loop:** Ensure seamless looping
- **Testing:** Send a test email to yourself first!

## 📧 Email Client Compatibility

| Email Client | Animated GIF Support |
|--------------|---------------------|
| Gmail        | ✅ Yes              |
| Outlook.com  | ✅ Yes              |
| Apple Mail   | ✅ Yes              |
| Outlook Desktop | ⚠️ Shows first frame only |
| Thunderbird  | ✅ Yes              |

**Note:** Outlook Desktop doesn't support animated GIFs, but recipients will still see your logo (just not animated).

## 🎯 Which Animation Should You Choose?

- **Professional Business:** Pop & Pulse
- **Creative/Marketing:** Bounce & Float
- **Tech/Startup:** Spin & Rotate
- **Sales/Bold:** Zoom & Breathe

## 📁 Files Included

```
├── logo.svg              # Pop & Pulse animation
├── logo-bounce.svg       # Bounce & Float animation
├── logo-rotate.svg       # Spin & Rotate animation
├── logo-zoom.svg         # Zoom & Breathe animation
├── animated-logo.html    # Single animation preview
├── preview-all.html      # Compare all animations
└── README.md            # This file
```

## 🛠️ Customization

To modify the animations, edit the SVG files. The animations are defined in the `<style>` section using CSS `@keyframes`.

### Key Animation Properties:
- **Duration:** Adjust the `animation` duration (e.g., `2s` for 2 seconds)
- **Delay:** Add delay before animation starts
- **Easing:** Change timing functions (`ease-in-out`, `cubic-bezier`, etc.)
- **Iterations:** Change `infinite` to a number for limited loops

## 📞 Need Help?

If you need to adjust colors, timing, or create a custom animation style, feel free to ask!

---

## 🔄 N8N Workflow: Google Sheets → Leadspeaker.com

### Co dělá tento workflow?

Automaticky synchronizuje kontakty z Google Sheets do platformy Leadspeaker.com:

1. ⏱️ **Monitoruje Google Sheets** - Sleduje nové řádky každou minutu
2. 🔄 **Mapuje data** - Převádí sloupce na správná pole
3. ☁️ **Nahrává kontakty** - Vytváří nové kontakty v Leadspeaker
4. 📊 **Loguje výsledky** - Zaznamenává úspěchy i chyby

### 🚀 Rychlý start

1. **Import workflow** do N8N:
   - Soubor: `google-sheets-to-leadspeaker.json`

2. **Konfigurace Google Sheets:**
   - Připojte Google účet
   - Vyberte spreadsheet
   - Nastavte trigger na "Row Added"

3. **Konfigurace Leadspeaker API:**
   - Získejte API klíč z Leadspeaker.com
   - Nastavte autorizační header v N8N

4. **Testování:**
   - Přidejte testovací řádek do Sheets
   - Zkontrolujte, že kontakt byl vytvořen v Leadspeaker

### 📚 Kompletní dokumentace

Detailní návod najdete v souboru **[N8N-WORKFLOW-SETUP.md](N8N-WORKFLOW-SETUP.md)**, který obsahuje:

- ✅ Krok za krokem instalaci
- ✅ Konfiguraci Google Sheets a Leadspeaker API
- ✅ Přizpůsobení mapování polí
- ✅ Řešení problémů
- ✅ Tipy a triky

### 📁 Soubory workflow

```
├── google-sheets-to-leadspeaker.json    # N8N workflow soubor
└── N8N-WORKFLOW-SETUP.md               # Kompletní dokumentace (CZ)
```

---

**Made with ❤️ for eye-catching email communications & marketing automation**
