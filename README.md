# Water today

A small water-intake tracker web app.

- Daily limit: 800 ml. The glass shows how much is left; past 800 ml it shows how much over.
- Quick add buttons: 5, 10, 15, 25, 30, 50, 100, 150 ml, plus a custom amount field.
- Today's log with remove and "Clear today"; resets automatically each new day.
- Data is saved in the browser's localStorage (per device, per browser).
- Design: light icy background (#F3F8FC), blue tones (#2A7DE1, #6EC3F0), deep ink (#0F2E4A).
  Fonts: Bricolage Grotesque (numbers/headings), Figtree (body).

## Files
- index.html: the whole app (HTML, CSS and JS in one file)
- manifest.json, sw.js, icon-192.png, icon-512.png: make it installable on Android
  ("Add to Home screen" / "Install app") and work offline once hosted over https
- WaterToday.exe: Windows build (Neutralinojs), unsigned

## Plan
Host on GitHub Pages (repo: water-tracker) so it can be installed on a phone.
