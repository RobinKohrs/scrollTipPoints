<script>
  import { tps } from "./assets/tp";
  import { csv } from "d3-fetch";
  import { geoOrthographic, geoNaturalEarth1, geoPath } from "d3-geo";
  import world from "./assets/world.json";
  import { fade, fly } from "svelte/transition";
  import { tweened } from "svelte/motion";

  const d3 = { geoOrthographic, geoNaturalEarth1, geoPath, csv };
  const globe = d3.geoNaturalEarth1();

  import IntroImages from "./introImages.svelte";
  import Warnings from "./warnings.svelte";

  import scrollama from "scrollama";
  import * as topojson from "topojson";
  const worldGJ = topojson.feature(world, world.objects.wworld);

  // data as csv
  let data_path = "./data/tp.csv";
  let data, dataLoaded;
  d3.csv(data_path).then((d) => {
    // sort data
    d.sort((cu, ne) => cu.min > ne.min);
    data = d;
    dataLoaded = true;
  });

  let currentIndex;

  let steps;
  let imgtext = "";
  let imgIndex;

  $: if (steps) {
    let imagescroller = scrollama();
    imagescroller
      .setup({
        step: ".introimg",
        offset: 0.3,
        progress: false,
      })
      .onStepEnter((e) => {
        imgIndex = e.index;
        switch (e.index) {
          case 0:
            imgtext = "We";
            break;
          case 1:
            imgtext = "We are";
            break;
          case 2:
            imgtext = "We are destroying";
            break;
          case 3:
            imgtext = "We are destroying <strong>Everything</strong>";
            break;
        }
      });

    let scroller = scrollama();
    scroller
      .setup({
        step: ".step",
        offset: 0.6,
        progress: true,
      })
      .onStepEnter((e) => {
        currentIndex = e.index;
      })
      .onStepProgress((e) => {})
      .onStepExit((e) => {
        if (e.index == 0 && e.direction == "up") {
          currentIndex = undefined;
        }
      });
  }

  let height, width;
  $: mobile = width <= 450;

  let projection;
  let path;

  $: if (height) {
    projection = globe.fitSize([svgSizes.width, svgSizes.height], {
      type: "Sphere",
    });
    path = d3.geoPath().projection(projection);
  }

  let options = {};
  let svgSizes = {};
  $: if (width) {
    svgSizes.width = width;
    svgSizes.height = height;
  }
  let x = tweened(0);
  let y = tweened(0);

  $: if (currentIndex >= 0) {
    let [xval, yval] = projection([
      +data[currentIndex].x,
      +data[currentIndex].y,
    ]);
    x.set(xval);
    y.set(yval);
  }
</script>

<svelte:window bind:innerHeight={height} bind:innerWidth={width} />

<IntroImages index={imgIndex} imgtext={imgtext.split(" ")} {mobile} />
<Warnings {currentIndex} disappear={currentIndex === 15} {mobile} />

{#if currentIndex !== undefined}
  <div
    in:fade={{ duration: 900 }}
    out:fade={{ duration: 900 }}
    class="header-panel"
    style="opacity: .8;"
  >
    <div class="min-panel">
      <div>Minimum</div>
      <div>
        {data[currentIndex].min} °C
      </div>
    </div>
    <div class="max-panel">
      <div>Maximum</div>
      <div>
        {data[currentIndex].max} °C
      </div>
    </div>
  </div>
{/if}
<div class="scrolly">
  <!-- The steps -->
  <div class="text-container" />
  <!-- the Map -->
  <div class="map-container">
    {#if height}
      <svg
        width={svgSizes.width}
        height={svgSizes.height}
        viewBox="0 0 {svgSizes.width} {svgSizes.height}"
      >
        {#each worldGJ.features as f, i}
          <g>
            <path d={path(f)} fill="#f8f8f8" stroke="#000" stroke-width=".5" />
          </g>
        {/each}
        {#if $x > 0 && $y > 0}
          <g>
            <filter id="blur">
              <feGaussianBlur stdDeviation="2" />
            </filter>
            <circle
              r={mobile ? "5" : "13"}
              fill="#ff0000"
              cx={$x}
              cy={$y}
              filter="url(#blur)"
            />
          </g>
        {/if}
      </svg>
    {/if}
  </div>
  <div class="scroll-container">
    {#if data}
      {#each data as step, i}
        <div
          class="step"
          style="z-index: 10;"
          class:active={i === currentIndex}
          style:height={height * 0.25 + "px"}
          bind:this={steps}
        >
          <div class="tp-name">
            {step.name}
          </div>
          <div class="tp-text">{@html step.content_en}</div>
        </div>
      {/each}
    {/if}
  </div>
</div>
<div class="text-container" style="padding-bottom: 400px; font-size: 4rem;">
  <div>This it it...</div>
</div>

<style lang="scss">
  :root {
    background-image: linear-gradient(lightblue, rgb(230, 175, 175));
  }

  .header-panel {
    position: fixed;
    display: flex;
    justify-content: space-around;
    top: 0;
    left: 0;
    right: 0;
    background-color: darkgray;
    padding: 1rem;
    color: black;
    font-size: 3rem;

    @media screen and(max-width: 450px) {
      font-size: 1rem;
    }

    & > div {
      display: flex;
      flex-direction: column;
    }
  }

  .text-container {
    max-width: 50vw;
    margin: 0 auto;
  }

  .scrolly {
    position: relative;
    z-index: -1000;

    & .map-container {
      position: sticky;
      height: 100vh;
      top: 0;
      z-index: 0;
      opacity: 0.8;
      // outline: 4px solid black;
    }

    & .scroll-container {
      position: relative;
      max-width: 50vw;
      margin: 0 auto;

      & .step {
        color: black;
        z-index: 1000;
        text-align: center;
        margin: 500px 0;
        opacity: 0.8;

        & > * {
          border: 1px solid black;
        }

        & .tp-name {
          background-color: #f1f1f1;
          font-size: 1.4rem;
          padding: 1rem;
          border-radius: 1rem 1rem 0 0;

          @media screen and(max-width: 450px) {
            font-size: 1rem;
            padding: 0.3rem;
          }
        }
        & .tp-text {
          background-color: #f1f1f1;
          border-radius: 0 0 1rem 1rem;
          padding: 0.2rem;

          @media screen and(max-width: 450px) {
            font-size: 0.8rem;
          }
        }
      }
      .step:last-child {
        padding-bottom: 400px;
        margin: 100px 0;
      }
    }
  }
</style>
