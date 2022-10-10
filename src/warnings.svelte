<script>
  import { fade } from "svelte/transition";
  import scrollama from "scrollama";

  let warnings = [
    "A new Study, published by ....",
    "Highlights <strong>16</strong> potential climate tipping points",
    "If they are reached,",
    "things might get out of control",
  ];
  let container;

  let index;
  let activeIndex;
  $: console.log(`active: `, activeIndex);
  $: if (container) {
    let scroller = scrollama();
    scroller
      .setup({
        step: ".warning-placeholder",
        offset: 0.3,
        progress: true,
        debug: true,
      })
      .onStepEnter((e) => {
        activeIndex = e.index;
      })
      .onStepExit((e) => {
        // activeIndex = e.index;
      })
      .onStepProgress((p) => {});
  }
</script>

<div class="container" bind:this={container}>
  <div style="position: fixed;">active: {activeIndex}</div>
  <div
    class="paintwall"
    style:height="{200 * warnings.length + 1}vh"
    style="background-color: black; position: relative; width: 100vw;"
  >
    {#each warnings as warning, i}
      <!-- absolutes -->
      <div
        class="warning-text"
        style:transform={"translate(-50%, -50%)"}
        style:scale={i === activeIndex ? 1 : 0}
        style="font-size: 3rem; position: fixed; left: 50%; top: 50%; color: white;
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
            class=""
            style:scale={i === activeIndex ? 1 : 0}
            style="font-size: 3rem; color:white; position: sticky; top: 50%; text-align: center;
            transition: all 1.4s;"
          >
            here
          </div>
        {/if}
      </div>
    {/each}
  </div>
</div>

<style lang="scss">
  .warning {
    // position: sticky;
    // outline: 1px solid black;
    // width: 100vw;
    // top: 0;
    // display: flex;
    // align-items: center;
    // justify-content: center;
    // font-size: 3rem;
    // background-image: linear-gradient(white, white);
    // background-size: cover;
    // background-repeat: no-repeat;
    // height: 100vh;
    // background-attachment: fixed;
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
</style>
