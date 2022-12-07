<script>
  // import logo from '../assets/logo192.png';
  import { createEventDispatcher, onMount, onDestroy } from 'svelte';
  // import logo from '../assets/RALOS1.svg';
  export let topSpeed = 5;
  export let minSpeed = 1;
  export let status = 'player';

  const dispatch = createEventDispatcher();

  let keys = {};
  export let size = 160;
  $: radius = size / 2;
  let maxSize = 400;
  $: xMax = -size + (() => window.innerWidth)();
  $: yMax = -size + (() => window.innerHeight)();
  let x = ~~(window.innerWidth / 2 - size / 2);
  let y = ~~(window.innerHeight / 2 - size / 2);
  const accel = .3;
  let rotation = 45;
  let flashDuration = 400;
  let flashingTimeout;

  let xStep = +(-topSpeed + Math.random() * (topSpeed * 2)).toFixed(2);
  if (xStep > -minSpeed && xStep < minSpeed) {
    xStep = xStep < 0 ? -minSpeed : minSpeed;
  }

  let yStep = +(-topSpeed + Math.random() * (topSpeed * 2)).toFixed(2);
  if (yStep > -minSpeed && yStep < minSpeed) {
    yStep = yStep < 0 ? -minSpeed : minSpeed;
  }

  export function resize() {
    const xPrevMax = xMax;
    const yPrevMax = yMax;
    xMax = window.innerWidth - size;
    yMax = window.innerHeight - size;
    const xConversion = xPrevMax / xMax;
    const yConversion = yPrevMax / yMax;
    x = ~~(x * xConversion);
    y = ~~(y * yConversion);
    if (x > xMax) x = xMax;
    if (y > yMax) y = yMax;
    return { x, y, size };
  }

  export function move() {
    x += xStep;
    y += yStep;
    if (x > xMax || x < 0) {
      xStep = -xStep;
      x = x < 0 ? 0 : xMax;
      x += xStep;
    }
    if (y > yMax || y < 0) {
      yStep = -yStep;
      y = y < 0 ? 0 : yMax;
      y += yStep;
    }
    rotation += xStep * 2;
    return { x, y, size, xStep, yStep };
  }

  function handleKeyDown({ key }) {
    if(key === 'Enter') return dispatch('enter-key');

    // if (keys[key]) return;
    keys[key] = true;
    switch(key) {
      case 'ArrowUp':
        keys.ArrowDown = undefined;
        break;
      case 'ArrowDown':
        keys.ArrowUp = undefined;
        break;
      case 'ArrowLeft':
        keys.ArrowRight = undefined;
        break;
      case 'ArrowRight':
        keys.ArrowLeft = undefined;
        break;
      default:
        break;
    }
    if (key === 'ArrowDown' || key === 'ArrowUp' || key === 'ArrowRight' || key === 'ArrowLeft') respondToInput();
  }

  function handleKeyUp({ key }) {
    // keys = {};
    keys[key] = undefined;
    // if (key === 'ArrowDown' || key === 'ArrowUp' || key === 'ArrowRight' || key === 'ArrowLeft') respondToInput();
  }

  function respondToInput() {
    if (keys.ArrowUp) {
      if (yStep > 0) {
        yStep *= (1 - accel);
        if (yStep < minSpeed) yStep = -minSpeed;
      } else if (yStep > -topSpeed) yStep *= (1 + accel);
    }
    if (keys.ArrowDown) {
      if (yStep < 0) {
        yStep *= (1 - accel);
        if (yStep > -minSpeed) yStep = minSpeed;
      } else if (yStep < topSpeed) yStep *= (1 + accel);
    }
    if(keys.ArrowRight) {
      if (xStep < 0) {
        xStep *= (1 - accel);
        if (xStep > -minSpeed) xStep = minSpeed;
      } else if (xStep < topSpeed) xStep *= (1 + accel);
    }
    if(keys.ArrowLeft) {
      if (xStep > 0) {
        xStep *= (1 - accel);
        if (xStep < minSpeed) xStep = -minSpeed;
      } else if (xStep > -topSpeed) xStep *= (1 + accel);
    }
  }

  // export function grow() {
  //   if (size > 16) {
  //     size --;
  //   }
  //   return { x, y, size, xStep, yStep };
  // }

  export function killConfirmed() {
    xMax = window.innerWidth - size;
    yMax = window.innerHeight - size;
    clearTimeout(flashingTimeout);
    status = 'flashing';
    flashingTimeout = setTimeout(() => status = 'player', flashDuration);
  }

  onMount(() => {
    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);
  });
  onDestroy(() => {
    window.removeEventListener('keydown', handleKeyDown);
    window.removeEventListener('keyup', handleKeyUp);
  });
</script>

<!-- <div class="player" class:flashing={status === 'flashing'} style="left:{x}px; top:{y}px; width:{size}px; height:{size}px; transform:rotate({rotation}deg);"></div> -->
  <!-- <img class="player" class:flashing={status === 'flashing'} src={logo} style="left:{x}px; top:{y}px; height:{size}px; width:{size}px; transform:rotate({rotation}deg);" alt="player" /> -->
  <svg
    version="1.1"
    viewBox="0 0 91.825066 90.454666"
    height="{size}px"
    width="{size}px"
    class="player"
    style="left:{x}px; top:{y}px; transform:rotate({rotation}deg);"
  >
    <g
      id="g10"
      transform="matrix(1.3333333,0,0,-1.3333333,-170.85852,650.38797)"
    >
      <g id="g12" transform="scale(0.1)">
        <g id="g772">
          <path
            id="path72"
            d="m 1841.0329,4803.3799 c -58.965,46.59 -133.777,74.53 -215.246,74.53 -190.176,0 -344.348,-151.87 -344.348,-339.21 0,-15.09 1.11,-29.93 3.047,-44.51 21.61,-130.87 136.418,-230.79 274.981,-230.79 31.812,0 62.336,5.36 90.812,15.05 -8.07,-0.72 -16.23,-1.14 -24.492,-1.14 -18.332,0 -36.223,1.84 -53.496,5.34 -120.855,24.16 -211.848,129.68 -211.848,256.2 0,18.39 1.946,36.33 5.602,53.65 25.797,146.08 154.652,257.16 309.891,257.16 60.578,0 117.101,-16.97 165.097,-46.28 v 0"
            fill="#f6bd0c"
          />
          <path
            id="path74"
            d="m 1412.7709,4272.2999 c 58.622,-45.54 132.563,-72.8 213.016,-72.8 190.172,0 344.34,151.87 344.34,339.2 0,3.02 -0.152,6 -0.234,9 -4.817,147.95 -127.637,266.45 -278.559,266.45 -31.813,0 -62.336,-5.36 -90.813,-15.05 8.075,0.73 16.231,1.15 24.5,1.15 18.325,0 36.215,-1.84 53.497,-5.34 120.851,-24.16 211.843,-129.67 211.843,-256.21 0,-18.39 -1.949,-36.33 -5.605,-53.65 -25.797,-146.07 -154.652,-257.15 -309.887,-257.15 -59.308,0 -114.758,16.22 -162.098,44.4 v 0"
            fill="#f47216"
          />
        </g>
      </g>
    </g>
  </svg>


<style>
  .player {
    z-index: 8;
    position:absolute;
  }
  .flashing {
    animation: linear infinite;
    animation-name: blinker;
    animation-duration: 100ms;
  }
  @keyframes blinker {
    50% {
      fill: white;
    }
  }
</style>
