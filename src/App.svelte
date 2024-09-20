<script lang="ts">
  import {onMount} from 'svelte';
  import { spring, tweened } from "svelte/motion";
  import {quadInOut} from 'svelte/easing';

  let firstIconEl, secondIconEl;
  const firstIconTranslate = spring(100, { stiffness: 0.2, damping: 0.1 });
  const secondIconTranslate = spring(100, { stiffness: 0.2, damping: 0.1 });

  function moveRacket(el) {
    if (el.target === firstIconEl) {
      firstIconTranslate.update(val => (val ? 0 : 100))
    } else if (el.target === secondIconEl) {
      secondIconTranslate.update(val => (val ? 0 : 100))
    } else {
      throw new Error('Invalid target clicked');
    }
  }

  const dotTranslateX = tweened(0, {
    duration: 1000,
    easing: quadInOut,
  })
  const dotTranslateY = tweened(0, {
    duration: 500,
    easing: quadInOut,
  });

  let ballAnimationEnabled = false;
  let goingForward = true; // The direction that the dot is traveling
  let dotArcingUp = true; // Switches to false halfway through the animation
  const arcHeight = 50; // Maximum height of the arc
  const distance = 360; // Distance traveled across x axis from start to end

  function animateBall() {
    if (ballAnimationEnabled === false) return;
    $: $dotTranslateX = $dotTranslateX === 0 ? distance : 0;
    $: $dotTranslateY = dotArcingUp === true ? -arcHeight : 0;
  }


  dotTranslateX.subscribe(x => {
    const atStart = x === 0;
    const pastHalfway = x > (distance / 2);
    const atEnd = x === distance;
    
    // If we're going forward and we hit the halfway mark, flip dotArcingUp
    // (but only if we haven't already done so)
    if (goingForward) {
      if (pastHalfway == true && dotArcingUp === true) {
        dotArcingUp = false;
        $dotTranslateY = 0;
      }
    } else {
      // if it's going backwards and hits midpoint, change arc to go down
      // (but only if we haven't already done so)
      if (pastHalfway === false && dotArcingUp === true) {
        dotArcingUp = false;
        $dotTranslateY = 0;
      }
    }

    // At the terminus of each direction, toggle direction & arc state.
    if (atStart || atEnd) {
      goingForward = atStart;
      dotArcingUp = true;
      animateBall();
    }
  });
</script>

<main>
  <h1>Racquet Animation</h1>
  <section class="animation-zone">
    <span
      id="first"
      bind:this={firstIconEl}
      on:click={moveRacket}
      class="material-symbols-outlined icon"
      style="transform: translateY(-{$firstIconTranslate}px)"
    >
      sports_tennis
    </span>
    <span
      class="dot"
      on:click={animateBall}
      style="transform: translateX({$dotTranslateX}px) translateY({$dotTranslateY}px)" 
    />
    <span 
      id="second"
      bind:this={secondIconEl}
      on:click={moveRacket}
      class="material-symbols-outlined icon"
      style="transform: translateY(-{$secondIconTranslate}px) scaleX(-1)"
    >
      sports_tennis
    </span>
  </section>
  <input type="checkbox" name="animating" bind:checked={ballAnimationEnabled} on:change={animateBall} />
  <label for="animating">Toggle animation</label>
</main>

<style>
  .animation-zone {
    height: 200px;
    background-color: #FFEDD8;
    padding: 1em;

    display: flex;
    align-items: end;
    justify-content: space-between;
    position: relative;
  }
  .icon {
    font-size: 2em;
    color: black;
    cursor: pointer;
    will-change: transform;
    user-select: none;
  }
  .dot {
    width: 10px;
    height: 10px;
    background-color: black;
    border-radius: 50%;
    cursor: pointer;

    position: absolute;
    left: 50px;
    bottom: 140px;
  }
</style>
