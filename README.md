<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Interactief rad</title>
  <style>
    :root {
      --green: #72df00;
      --orange: #ff8a45;
      --blue: #4c66f0;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      min-height: 100svh;
      display: grid;
      place-items: center;
      background: #f7f7f7;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      color: #1d1d1d;
    }

    .app {
      width: min(1120px, 96vw);
      display: grid;
      grid-template-columns: minmax(310px, 760px) 180px;
      gap: clamp(20px, 4vw, 54px);
      align-items: center;
      padding: 24px;
    }

    .wheel-stage {
      position: relative;
      width: min(760px, 88vw);
      aspect-ratio: 1;
      margin-inline: auto;
    }

    .wheel-rotor {
      position: absolute;
      inset: 0;
      transform-origin: 50% 50%;
      transform: rotate(0deg);
      transition: transform 3.8s cubic-bezier(.12,.72,.08,1);
      filter: drop-shadow(0 16px 30px rgba(0,0,0,.18));
    }

    .wheel-image,
    .emphasis-svg {
      position: absolute;
      inset: 0;
      width: 100%;
      height: 100%;
      display: block;
    }

    .wheel-image {
      user-select: none;
      -webkit-user-drag: none;
    }

    .emphasis-svg {
      pointer-events: none;
      overflow: visible;
    }

    .fade-slice,
    .active-ring {
      opacity: 0;
      transition: opacity .55s ease;
    }

    .wheel-rotor.settled .fade-slice.visible {
      opacity: .56;
    }

    .wheel-rotor.settled .active-ring.visible {
      opacity: 1;
    }

    .hub-shine {
      position: absolute;
      left: 50%;
      top: 50%;
      width: 15%;
      height: 15%;
      transform: translate(-50%, -50%);
      border-radius: 999px;
      pointer-events: none;
      box-shadow: 0 0 0 3px rgba(255,255,255,.35), 0 0 26px rgba(255,255,255,.45);
      z-index: 3;
      opacity: 0;
      transition: opacity .55s ease;
    }

    .wheel-rotor.settled + .hub-shine {
      opacity: 1;
    }

    .pointer {
      position: absolute;
      z-index: 5;
      top: -6px;
      left: 50%;
      transform: translateX(-50%);
      width: 0;
      height: 0;
      border-left: 25px solid transparent;
      border-right: 25px solid transparent;
      border-top: 0;
      border-bottom: 46px solid #1f1f1f;
      filter: drop-shadow(0 5px 4px rgba(0,0,0,.25));
    }

    .controls {
      display: grid;
      gap: 16px;
    }

    button {
      border: 0;
      border-radius: 22px;
      padding: 20px 22px;
      font-size: 1.08rem;
      font-weight: 800;
      cursor: pointer;
      box-shadow: 0 10px 24px rgba(0,0,0,.18);
      transition: transform .15s ease, filter .15s ease, opacity .15s ease;
    }

    button:hover {
      transform: translateY(-2px);
      filter: brightness(1.04);
    }

    button:active {
      transform: translateY(1px);
    }

    button:disabled {
      cursor: wait;
      opacity: .7;
      transform: none;
    }

    .btn-green {
      background: var(--green);
      color: #174800;
    }

    .btn-orange {
      background: var(--orange);
      color: #fff;
    }

    .btn-blue {
      background: var(--blue);
      color: #fff;
    }

    .status {
      min-height: 1.4em;
      margin: 10px 0 0;
      text-align: center;
      color: #555;
      font-size: .95rem;
    }

    @media (max-width: 820px) {
      .app {
        grid-template-columns: 1fr;
        gap: 22px;
      }

      .controls {
        grid-template-columns: repeat(3, 1fr);
      }

      button {
        padding: 16px 10px;
        border-radius: 18px;
      }
    }
  </style>
</head>
<body>
  <main class="app">
    <section class="wheel-stage" aria-label="Interactief rad">
      <div class="pointer" aria-hidden="true"></div>

      <div id="rotor" class="wheel-rotor">
        <img class="wheel-image" src="assets/rad.png" alt="Rad met groene, oranje en blauwe vakken">

        <!-- Deze SVG ligt exact over rad.png heen.
             De witte vlakken maken de niet-gekozen sectoren transparanter.
             De ring geeft het gekozen vak extra nadruk. -->
        <svg class="emphasis-svg" viewBox="0 0 600 600" aria-hidden="true">
          <defs>
            <filter id="softGlow" x="-30%" y="-30%" width="160%" height="160%">
              <feGaussianBlur stdDeviation="5" result="blur"/>
              <feMerge>
                <feMergeNode in="blur"/>
                <feMergeNode in="SourceGraphic"/>
              </feMerge>
            </filter>
          </defs>

          <path class="fade-slice fade-green" data-color="green"
                d="M300 300 L52.3 157 A286 286 0 0 1 547.7 157 Z"
                fill="#ffffff"/>
          <path class="fade-slice fade-blue" data-color="blue"
                d="M300 300 L547.7 157 A286 286 0 0 1 300 586 Z"
                fill="#ffffff"/>
          <path class="fade-slice fade-orange" data-color="orange"
                d="M300 300 L300 586 A286 286 0 0 1 52.3 157 Z"
                fill="#ffffff"/>

          <path class="active-ring ring-green" data-color="green"
                d="M300 300 L52.3 157 A286 286 0 0 1 547.7 157 Z"
                fill="none" stroke="#ffffff" stroke-width="15" filter="url(#softGlow)"/>
          <path class="active-ring ring-blue" data-color="blue"
                d="M300 300 L547.7 157 A286 286 0 0 1 300 586 Z"
                fill="none" stroke="#ffffff" stroke-width="15" filter="url(#softGlow)"/>
          <path class="active-ring ring-orange" data-color="orange"
                d="M300 300 L300 586 A286 286 0 0 1 52.3 157 Z"
                fill="none" stroke="#ffffff" stroke-width="15" filter="url(#softGlow)"/>
        </svg>
      </div>

      <div class="hub-shine" aria-hidden="true"></div>
    </section>

    <aside>
      <div class="controls" aria-label="Kies een kleur">
        <button class="btn-green" data-target="green">Groen</button>
        <button class="btn-orange" data-target="orange">Oranje</button>
        <button class="btn-blue" data-target="blue">Blauw</button>
      </div>
      <p id="status" class="status" aria-live="polite"></p>
    </aside>
  </main>

  <script>
    const rotor = document.querySelector("#rotor");
    const buttons = document.querySelectorAll("button[data-target]");
    const status = document.querySelector("#status");

    // Startpositie van rad.png:
    // groen staat midden boven, blauw rechts onder, oranje links onder.
    const targetRotation = {
      green: 0,
      blue: -120,
      orange: 120
    };

    const colorLabel = {
      green: "groen",
      blue: "blauw",
      orange: "oranje"
    };

    let currentRotation = 0;
    let isSpinning = false;

    function highlight(color) {
      document.querySelectorAll(".fade-slice").forEach(slice => {
        slice.classList.toggle("visible", slice.dataset.color !== color);
      });

      document.querySelectorAll(".active-ring").forEach(ring => {
        ring.classList.toggle("visible", ring.dataset.color === color);
      });

      rotor.classList.add("settled");
      status.textContent = `Het rad staat op ${colorLabel[color]}.`;
    }

    function spinTo(color) {
      if (isSpinning) return;

      isSpinning = true;
      rotor.classList.remove("settled");
      status.textContent = "Het rad draait…";
      buttons.forEach(button => button.disabled = true);

      const baseTarget = targetRotation[color];

      const fullTurns = 4 + Math.floor(Math.random() * 2);
      const normalizedCurrent = ((currentRotation % 360) + 360) % 360;
      const normalizedTarget = ((baseTarget % 360) + 360) % 360;

      let delta = normalizedTarget - normalizedCurrent;
      if (delta < 0) delta += 360;

      currentRotation += fullTurns * 360 + delta;
      rotor.style.transform = `rotate(${currentRotation}deg)`;

      window.setTimeout(() => {
        highlight(color);
        buttons.forEach(button => button.disabled = false);
        isSpinning = false;
      }, 3900);
    }

    buttons.forEach(button => {
      button.addEventListener("click", () => spinTo(button.dataset.target));
    });

    highlight("green");
  </script>
</body>
</html>
