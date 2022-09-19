<script>
  import { tps } from "./assets/tp";
  import { geoOrthographic, geoNaturalEarth1, geoPath } from "d3-geo";
  import world from "./assets/world.json";
  import { fade, fly } from "svelte/transition";
  import { tweened } from "svelte/motion";

  const d3 = { geoOrthographic, geoNaturalEarth1, geoPath };
  const globe = d3.geoNaturalEarth1();

  import scrollama from "scrollama";
  import * as topojson from "topojson";
  const worldGJ = topojson.feature(world, world.objects.wworld);

  let mapContainer, steps;
  let scroller;
  // initialize the scrollama

  let currentIndex;
  $: console.log(`currentIndex: `, currentIndex);
  $: if (mapContainer) {
    scroller = scrollama();
    scroller
      .setup({
        step: ".step",
        offset: 0.6,
        progress: true,
        // debug: true,
      })
      .onStepEnter((e) => {
        currentIndex = e.index;
      })
      .onStepProgress((e) => {});
  }

  let height, width;

  let projection;
  let path;

  $: if (height) {
    projection = globe.fitSize([svgSizes.width, svgSizes.height], {
      type: "Sphere",
    });
    console.log("projection: ", projection);
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

  $: console.log(`xCoord: `, x);
  $: if (currentIndex >= 0) {
    let [xval, yval] = projection(tps[currentIndex].coords);
    x.set(xval);
    y.set(yval);
  }
</script>

<svelte:window bind:innerHeight={height} bind:innerWidth={width} />

<div class="scrolly">
  <!-- The steps -->
  <div class="text-container">
    Lorem ipsum dolor, sit amet consectetur adipisicing elit. Repellendus
    necessitatibus eveniet, aperiam et autem, quas quasi omnis soluta quos
    maiores, commodi at! Porro ipsa quaerat nostrum mollitia qui cumque
    corrupti. Pariatur autem natus porro assumenda, tempora exercitationem omnis
    mollitia. Distinctio minima nisi temporibus recusandae voluptate iste
    voluptates doloribus, vitae consequatur dolorum eligendi praesentium
    repellat earum autem inventore impedit. Officiis, quasi. Autem nulla debitis
    optio inventore dicta atque, ea, reprehenderit ut non commodi quae ipsam?
    Labore ducimus, temporibus harum officiis repudiandae qui consequatur,
    expedita deserunt eos ea similique maiores iure veniam. Voluptatem eum quo,
    quidem perferendis at neque quod quisquam modi voluptatum odio quia sed,
    animi, reiciendis magni praesentium sapiente ducimus perspiciatis a sit ex
    itaque. Similique reprehenderit reiciendis atque dolorem. Ducimus voluptatem
    harum non dolore necessitatibus deleniti soluta ullam magni, numquam
    nesciunt delectus cum, aliquam officiis amet magnam! Placeat at earum
    veritatis laborum. Assumenda alias sed omnis deleniti sint laudantium.
  </div>
  <!-- the Map -->
  <div class="map-container" bind:this={mapContainer}>
    {#if height}
      <svg
        width={svgSizes.width}
        height={svgSizes.height}
        viewBox="0 0 {svgSizes.width} {svgSizes.height}"
      >
        {#each worldGJ.features as f}
          <g>
            <path
              d={path(f)}
              fill="#f8f8f8"
              stroke="#ababab"
              stroke-width="0.5"
            />
          </g>
        {/each}
        {#if $x > 0 && $y > 0}
          <g>
            <filter id="blur">
              <feGaussianBlur stdDeviation="2" />
            </filter>
            <circle r="13" fill="#ff0000" cx={$x} cy={$y} filter="url(#blur)" />
          </g>
        {/if}
      </svg>
    {/if}
  </div>
  <div class="text-container">
    {#each tps as step, i}
      <div
        class="step"
        class:active={i === currentIndex}
        style:height={height * 0.25 + "px"}
      >
        <div class="tp-name">
          {step.name}
        </div>
        <div class="tp-text">
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Sit
          voluptatum quod blanditiis libero assumenda, debitis temporibus.
          Similique totam sequi quam blanditiis rem facere natus, voluptatum nam
          sunt officiis et inventore!
        </div>
      </div>
    {/each}
  </div>
</div>

<style lang="scss">
  :root {
    background-image: linear-gradient(
      rgba(173, 216, 230, 0.1),
      // rgba(255, 0, 0, 0.2)
    );
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

    & .text-container {
      position: relative;
      max-width: 50vw;
      margin: 0 auto;

      & .step {
        color: black;
        z-index: 1000;
        text-align: center;
        margin: 500px 0;
        // font-size: 2px;
        opacity: 0.1;
        display: flex;
        flex-direction: column;
        align-items: center;
      }

      & .step .tp-name {
        background-color: black;
        background-color: rgba($color: #000000, $alpha: 0.2);
        border-radius: 1rem;
        margin: 1rem;
        padding: 1rem;
        max-width: fit-content;
      }

      & .step.active {
        padding: 20px;
        opacity: 1;
        font-size: 1rem;
        transition: all 0.5s;
      }

      & .step:last-child {
        padding-bottom: 1000px;
      }
    }
  }
</style>
