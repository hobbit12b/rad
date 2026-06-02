<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Interactief rad</title>
  <style>
    :root {
      --green: #67d900;
      --orange: #ff914d;
      --blue: #4f6df3;
      --panel: #ffffff;
      --text: #1f2937;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      background: #f4f6fb;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      color: var(--text);
    }

    .app {
      width: min(1120px, 96vw);
      display: grid;
      grid-template-columns: minmax(300px, 680px) 180px;
      gap: 32px;
      align-items: center;
      justify-content: center;
      padding: 28px;
    }

    .wheel-card {
      position: relative;
      width: min(680px, 86vw);
      aspect-ratio: 1 / 1;
      display: grid;
      place-items: center;
    }

    .wheel-wrap {
      position: relative;
      width: 100%;
      height: 100%;
      border-radius: 50%;
      overflow: hidden;
      filter: drop-shadow(0 18px 30px rgba(0,0,0,.18));
    }

    .wheel {
      width: 100%;
      height: 100%;
      display: block;
      border-radius: 50%;
      transform: rotate(0deg);
      transition: transform 4s cubic-bezier(.12,.72,.12,1);
      user-select: none;
      -webkit-user-drag: none;
    }

    /* Na het stoppen: het bovenste vak blijft helder, de twee andere vakken worden gedimd. */
    .wheel-wrap.stopped::after {
      content: "";
      position: absolute;
      inset: 0;
      border-radius: 50%;
      pointer-events: none;
      background: conic-gradient(
        from -60deg at 50% 50%,
        rgba(255,255,255,0) 0deg 120deg,
        rgba(255,255,255,.58) 120deg 360deg
      );
    }

    .wheel-wrap.stopped::before {
      content: "";
      position: absolute;
      inset: 0;
      border-radius: 50%;
      pointer-events: none;
      background: conic-gradient(
        from -60deg at 50% 50%,
        rgba(255,255,255,.38) 0deg 120deg,
        rgba(255,255,255,0) 120deg 360deg
      );
      mix-blend-mode: screen;
    }

    .pointer {
      position: absolute;
      top: -5px;
      left: 50%;
      transform: translateX(-50%);
      width: 0;
      height: 0;
      border-left: 24px solid transparent;
      border-right: 24px solid transparent;
      border-top: 42px solid #111827;
      z-index: 4;
      filter: drop-shadow(0 5px 5px rgba(0,0,0,.25));
    }

    .buttons {
      display: grid;
      gap: 16px;
    }

    button {
      min-height: 64px;
      border: 0;
      border-radius: 18px;
      color: white;
      font-size: 1.15rem;
      font-weight: 800;
      cursor: pointer;
      box-shadow: 0 10px 20px rgba(0,0,0,.18);
      transition: transform .16s ease, filter .16s ease, opacity .16s ease;
    }

    button:hover { transform: translateY(-2px); filter: brightness(1.04); }
    button:active { transform: translateY(1px) scale(.99); }
    button:disabled { cursor: wait; opacity: .7; transform: none; }

    .green { background: var(--green); }
    .orange { background: var(--orange); }
    .blue { background: var(--blue); }

    .status {
      grid-column: 1 / -1;
      text-align: center;
      min-height: 1.5em;
      font-weight: 700;
    }

    @media (max-width: 760px) {
      .app {
        grid-template-columns: 1fr;
        gap: 22px;
        padding: 18px;
      }

      .buttons {
        grid-template-columns: repeat(3, 1fr);
      }

      button {
        min-height: 56px;
        font-size: 1rem;
        border-radius: 14px;
      }
    }
  </style>
</head>
<body>
  <main class="app">
    <section class="wheel-card" aria-label="Interactief rad">
      <div class="pointer" aria-hidden="true"></div>
      <div class="wheel-wrap" id="wheelWrap">
        <img class="wheel" id="wheel" src="assets/rad.png" alt="Rad met groene, oranje en blauwe vakken" />
      </div>
    </section>

    <nav class="buttons" aria-label="Kies een kleur">
      <button class="green" type="button" data-color="green">Groen</button>
      <button class="orange" type="button" data-color="orange">Oranje</button>
      <button class="blue" type="button" data-color="blue">Blauw</button>
    </nav>

    <div class="status" id="status" aria-live="polite"></div>
  </main>

  <script>
    const wheel = document.getElementById('wheel');
    const wheelWrap = document.getElementById('wheelWrap');
    const status = document.getElementById('status');
    const buttons = [...document.querySelectorAll('button[data-color]')];

    // Doelposities in graden.
    // Het rad.png heeft groen al bovenaan. Oranje en blauw worden naar boven gedraaid.
    const targetRotation = {
      green: 0,
      orange: 120,
      blue: 240
    };

    const label = {
      green: 'groen',
      orange: 'oranje',
      blue: 'blauw'
    };

    let currentRotation = 0;
    let isSpinning = false;

    function setButtonsDisabled(disabled) {
      buttons.forEach(button => button.disabled = disabled);
    }

    function spinTo(color) {
      if (isSpinning) return;

      isSpinning = true;
      setButtonsDisabled(true);
      wheelWrap.classList.remove('stopped');
      status.textContent = 'Het rad draait...';

      const fullTurns = 4 * 360;
      const currentMod = ((currentRotation % 360) + 360) % 360;
      const target = targetRotation[color];
      const delta = (target - currentMod + 360) % 360;

      currentRotation += fullTurns + delta;
      wheel.style.transform = `rotate(${currentRotation}deg)`;

      window.setTimeout(() => {
        wheelWrap.classList.add('stopped');
        status.textContent = `Het rad staat op ${label[color]}.`;
        isSpinning = false;
        setButtonsDisabled(false);
      }, 4100);
    }

    buttons.forEach(button => {
      button.addEventListener('click', () => spinTo(button.dataset.color));
    });
  </script>
</body>
</html>
