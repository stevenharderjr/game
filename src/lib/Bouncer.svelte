<script>
  import { createEventDispatcher, onMount, onDestroy } from 'svelte';
  import { vector, reflect, force } from '../utils/vector.js'
  const dispatch = createEventDispatcher();

  export let player = { x: -100, y: -100, size: 0 };
  export let size = 160;
  export let topSpeed = 5;
  export let minSpeed = 1;
  export let id;
  let boostLimit = 1.5;
  let maxSize = 160;
  let growthInterval;
  let growthDelay = 250;

  $: xMax = -size + (() => window.innerWidth)();
  $: yMax = -size + (() => window.innerHeight)();
  let alive = true;
  let x = ~~(Math.random() * window.innerWidth - size);
  let y = ~~(Math.random() * window.innerHeight - size);
  let accel = .2;

  let xStep = +(-topSpeed + Math.random() * (topSpeed * 2)).toFixed(2);
  if (xStep > -minSpeed && xStep < minSpeed) {
    xStep = xStep < 0 ? -minSpeed : minSpeed;
  }

  let yStep = +(-topSpeed + Math.random() * (topSpeed * 2)).toFixed(2);
  if (yStep > -minSpeed && yStep < minSpeed) {
    yStep = yStep < 0 ? -minSpeed : minSpeed;
  }

  export function init() {
    clearInterval(growthInterval);
    alive = true;
    x = ~~(Math.random() * window.innerWidth - size);
    y = ~~(Math.random() * window.innerHeight - size);
  }

  export function resize() {
    const { innerWidth, innerHeight } = window;
    const xRatio = innerWidth / xMax;
    const yRatio = innerHeight / yMax;
    x *= xRatio;
    y *= yRatio;
    xMax = innerWidth;
    yMax = innerHeight;
  }

  export function move() {
    if (!alive) return;

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
    const { x: playerX, y: playerY, size: playerSize, xStep: h, yStep: v } = player;
    if (playerSize) {
      if (x + size < playerX + playerSize * .2) return;
      if (y + size < playerY + playerSize * .2) return;
      if (x > playerX + playerSize * .8) return;
      if (y > playerY + playerSize * .8) return;
      const bouncer = { x, y };
      // direction of bouncer to player
      const positionVector = vector({ x: playerX + playerSize / 2, y: playerY + playerSize / 2 }, { x: x + size / 2, y: y + size / 2 });
      // player trajectory
      const playerVector = vector(player, { h, v });
      // bouncer trajectory
      const bouncerVector = vector(bouncer, { h: xStep, v: yStep });
      // median of player and bouncer trajectories?
      const newVector = vector(playerVector, bouncerVector);
      // median of median trajectory and position direction
      const combinedVector = vector(newVector, positionVector);
      // reflection of trajectory/position vector
      dispatch('bounce', combinedVector);
      const bounceVector = force(reflect(combinedVector), .8);
      // const bounceVector = vector(positionVector, bouncerVector);
      xStep = bounceVector.x;
      yStep = bounceVector.y;
      x += xStep;
      y += yStep;

      if (xStep < 0) {
        const min = -minSpeed;
        const max = -topSpeed * boostLimit;
        if (xStep > min) xStep = min;
        if (xStep < max) xStep = max;
      } else {
        const max = topSpeed * boostLimit;
        if (xStep < minSpeed) xStep = minSpeed;
        if (xStep > max) xStep = max;
      }

      if (yStep < 0) {
        const min = -minSpeed;
        const max = -topSpeed * boostLimit;
        if (yStep > min) yStep = min;
        if (yStep < max) yStep = max;
      } else {
        const max = topSpeed * boostLimit;
        if (yStep < minSpeed) yStep = minSpeed;
        if (yStep > max) yStep = max;
      }
    }
  }
</script>

<div class="bouncer" class:dead={!alive} style="left:{x}px; top:{y}px; height:{size}px; width:{size}px; border-radius:{size / 2}px"></div>

<style>
  .bouncer {
    position:absolute;
    background:#000;
    z-index: 1;
  }
  .dead {
    display: hidden;
  }
</style>
