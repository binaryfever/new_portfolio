<svg
  viewBox="0 0 400 400"
  width="100%"
  height="100%"
  role="img"
  aria-labelledby="skills-anim-title"
>
  <title id="skills-anim-title">Abstract diagram of interconnected skill domains: Design, Engineering, and Leadership</title>

  <defs>
    <path id="edge-path-1" d="M 200 60 Q 270 170 321 270" />
    <path id="edge-path-2" d="M 321 270 Q 200 290 79 270" />
    <path id="edge-path-3" d="M 79 270 Q 130 170 200 60" />
  </defs>

  <!-- Layer 1: Connecting edges -->
  <g class="edges">
    <path class="edge edge--1" d="M 200 60 Q 270 170 321 270" pathLength="1" />
    <path class="edge edge--2" d="M 321 270 Q 200 290 79 270" pathLength="1" />
    <path class="edge edge--3" d="M 79 270 Q 130 170 200 60" pathLength="1" />
  </g>

  <!-- Layer 2: Domain nodes -->
  <g class="nodes">
    <circle class="node" cx="200" cy="60"  r="20" />
    <circle class="node" cx="321" cy="270" r="20" />
    <circle class="node" cx="79"  cy="270" r="20" />
    <circle class="node node--center" cx="200" cy="200" r="12" />
  </g>

  <!-- Layer 3: Orbital particles -->
  <g class="particles">
    <circle class="particle particle--1" r="4" />
    <circle class="particle particle--2" r="4" />
    <circle class="particle particle--3" r="4" />
  </g>
</svg>

<style>
  /* ─── Edges ──────────────────────────────────────────────────────────── */
  .edge {
    fill: none;
    stroke: var(--color-edge-stroke);
    stroke-width: 1.5;
    stroke-dasharray: 0.08 0.92;
    stroke-dashoffset: 0;
    animation: edgeTravel var(--duration-edge-travel) var(--ease-linear) infinite;
  }

  .edge--2 { animation-delay: calc(var(--duration-edge-travel) / -3); }
  .edge--3 { animation-delay: calc(var(--duration-edge-travel) * -2 / 3); }

  @keyframes edgeTravel {
    to { stroke-dashoffset: -1; }
  }

  /* ─── Nodes ──────────────────────────────────────────────────────────── */
  .node {
    fill: var(--color-node-fill);
    stroke: var(--color-node-stroke);
    stroke-width: 1.5;
  }

  .node--center {
    fill: var(--color-center-node);
    stroke: none;
    opacity: var(--opacity-center-static);
    animation: pulse var(--duration-pulse) ease-in-out infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: var(--opacity-center-static); }
    50%       { opacity: 1; }
  }

  /* ─── Particles ──────────────────────────────────────────────────────── */
  .particle {
    fill: var(--color-particle);
  }

  .particle--1 {
    offset-path: url(#edge-path-1);
    offset-distance: 0%;
    animation: orbit var(--duration-orbit-1) var(--ease-linear) infinite;
  }

  .particle--2 {
    offset-path: url(#edge-path-2);
    offset-distance: 0%;
    animation: orbit var(--duration-orbit-2) var(--ease-linear) infinite;
    animation-delay: calc(var(--duration-orbit-2) / -2);
  }

  .particle--3 {
    offset-path: url(#edge-path-3);
    offset-distance: 0%;
    animation: orbit var(--duration-orbit-3) var(--ease-linear) infinite;
    animation-delay: calc(var(--duration-orbit-3) / -3);
  }

  @keyframes orbit {
    from { offset-distance: 0%; }
    to   { offset-distance: 100%; }
  }

  /* ─── Reduced Motion ─────────────────────────────────────────────────── */
  @media (prefers-reduced-motion: reduce) {
    .edge {
      animation: none;
      stroke-dasharray: none;
      stroke-dashoffset: 0;
      opacity: 1;
    }

    .node--center {
      animation: none;
      opacity: var(--opacity-center-static);
    }

    .particle {
      animation: none;
      offset-distance: 0%;
    }
  }
</style>
