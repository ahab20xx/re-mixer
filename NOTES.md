# re-mixer architecture 

re-mixer is a live audio and video effects web app built with HTML, CSS and JavaScript. Key points:

- Single index.html file containing all markup, styles and code
- Audio engine: Web Audio API with custom AudioWorklet effects
- Video engine: HTML video element with 2D canvas effects applied each frame
- Module panels: modular UI with resizable, rearrangeable effect controls
- Effect code: each effect is a self-contained JS object with parameters, rendering, etc.
- Settings storage: all effect parameters saved to localStorage
- Chaos toy: apply random parameter mutations by dragging the cube icon
- Video viewer: scalable live preview with audio oscilloscope and TV effect options
- Pads: XY touch surfaces that control effect parameters in real-time
- Audio meters: live level meters showing effect gain staging
- Recording: locally record audio+video with all effects applied
- Offline testing: Playwright+Chrome headless test harness for smoke testing changes

## Dev workflow

1. Clone the repo and open index.html directly in Chrome or Edge (Firefox blocks some features when run from file://)

2. Make changes in your code editor

3. Reload the page to see changes - no build step needed! 

4. Use the built-in record feature to capture audio+video of your jam

5. When ready, commit, push to GitHub, and copy index.html to your custom URL folder on your machine so Claude can reload it

Some tips:
- CTRL+SHIFT+I opens Chrome dev tools to see console logs and tweak things live
- You can connect a MIDI controller to automate the XY pads or buttons 
- To customize further, edit the effect functions directly in index.html (search for e.g. "bubbleTick" or "acidTick")

## History

re-mixer started as an experiment in seeing how much interactive audio and video could be squeezed into a single HTML file with no build steps, using modern web APIs. It's inspired by Roland SP-404 and other hardware effects boxes. The goal was to make a tactile, performative UI for mangling samples that's easy to modify and extend.

## Acknowledgments 

re-mixer makes use of:
- [Meyda](https://meyda.js.org/) for audio feature extraction
- [lamejs](https://github.com/zhuker/lamejs) for MP3 encoding
- [Playwright](https://playwright.dev/) for smoke testing

re-mixer is not affiliated with or endorsed by any company.

Let me know if you have any other questions! And most importantly - have fun remixing!
