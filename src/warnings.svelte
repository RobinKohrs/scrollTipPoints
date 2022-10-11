<script>
  export let currentIndex;
  export let disappear;
  export let mobile;
  $: console.log(`mobile`, mobile);

  import { fade } from "svelte/transition";
  import scrollama from "scrollama";

  //   D3
  import { csv } from "d3-fetch";
  import { geoOrthographic, geoNaturalEarth1, geoPath } from "d3-geo";
  const d3 = { geoOrthographic, geoNaturalEarth1, geoPath, csv };
  const globe = d3.geoNaturalEarth1();

  //   World
  import world from "./assets/world.json";
  import * as topojson from "topojson";
  import { xlink_attr } from "svelte/internal";
  const worldGJ = topojson.feature(world, world.objects.wworld);

  // TPs
  let data_path = "./data/tp.csv";
  let data, dataLoaded;
  d3.csv(data_path).then((d) => {
    // sort data
    d.sort((cu, ne) => cu.min > ne.min);
    data = d;
    dataLoaded = true;
  });

  let projection;
  let path;
  let height, width;
  $: if (height) {
    projection = globe.fitSize([width, height], {
      type: "Sphere",
    });
    path = d3.geoPath().projection(projection);
  }

  let warnings = [
    'A new Study, published by scientist around <a href="https://www.science.org/doi/10.1126/science.abn7950">Mc. Kay et al. (2022)</a>',
    "Highlights <strong>16</strong> potential climate tipping points",
    "that if reached",
    "might put things out of our control...",
  ];
  let container;

  let index;
  let activeIndex;
  let shake;
  $: if (container) {
    let scroller = scrollama();
    scroller
      .setup({
        step: ".warning-placeholder",
        offset: 0.3,
        progress: true,
        debug: false,
      })
      .onStepEnter((e) => {
        activeIndex = e.index;
        shake = false;
      })
      .onStepExit((e) => {
        if (e.index === 0) activeIndex = undefined;
        if (e.index === 3) shake = true;
      })
      .onStepProgress((p) => {});
  }
</script>

<svelte:window bind:innerHeight={height} bind:innerWidth={width} />

<div class="container" bind:this={container}>
  <div style="position: fixed;">active: {activeIndex}</div>
  <div class="map" style="position: fixed; top: 0; left: 0; z-index: 0;">
    <svg {width} {height}>
      {#if data && activeIndex !== undefined && !disappear}
        {#each data as d, i}
          {@const [x, y] = projection([+d.x, +d.y])}
          <g>
            <filter id="blur">
              <feGaussianBlur stdDeviation="2" />
            </filter>
            <circle
              in:fade={{ duration: 1200 }}
              r={mobile ? "5" : "13"}
              fill={i <= currentIndex ? "#000000" : "#ff0000"}
              cx={x}
              cy={y}
              filter="url(#blur)"
            />
          </g>
        {/each}
      {/if}
    </svg>
  </div>
  <div
    class="paintwall"
    style:height="{200 * warnings.length + 1}vh"
    style="background-color: black; position: relative; width: 100vw; z-index: -1"
  >
    {#each warnings as warning, i}
      <!-- absolutes -->
      <div
        class="warning-text"
        style:transform={"translate(-50%, -50%)"}
        style:font-size={mobile ? "2rem" : "3rem"}
        style:scale={i === activeIndex ? 1 : 0}
        style="z-index: 20; position: fixed; left: 50%; top: 50%; color: white;
          transition: all 1.4s"
      >
        {#if i != warnings.length - 1}
          {@html warning}
        {/if}
      </div>

      <!-- relatives -->
      <div
        class="warning-placeholder"
        style="height: 200vh; color: white; width: 100vw; position: relative"
      >
        {#if i === warnings.length - 1}
          <div
            class:shake
            style:scale={i === activeIndex ? 1 : 0}
            style="font-size: 3rem; color:white; position: sticky; top: 50%; text-align: center;
            transition: all 1.4s;"
          >
            {@html warning}
            {#if shake}
              <audio autoplay>
                <source src="sound/thunder.wav" type="audio/x-wav" />
              </audio>
            {/if}
          </div>
        {/if}
      </div>
    {/each}
  </div>
</div>

<style lang="scss">
  .warning {
    width: 100vw;
    position: relative;
    background-color: black;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;

    & .warning-text {
      position: absolute;
      color: white;
    }
  }

  .shake {
    animation: shake 0.5s;
  }

  @keyframes shake {
    0% {
      transform: translateX(0);
    }
    10% {
      transform: translateX(25px);
    }
    14% {
      transform: translateX(25px);
    }
    19% {
      transform: translateX(25px);
    }
    25% {
      transform: translateX(20px);
    }
    25% {
      transform: translateY(20px);
      transform: translateY(-20px);
    }
    30% {
      transform: translateX(10px);
      transform: translateY(40px);
    }
    35% {
      transform: translateX(40px);
    }

    50% {
      transform: translateX(-25px);
    }
    100% {
      transform: translateX(0px);
    }
  }
</style>
