<script>
  import { fade } from "svelte/transition";
  import scrollama from "scrollama";

  let warnings = [
    "Weather is changing fast",
    "Climate is changing slowly",
    "We are used to the rush",
    "And don't see climate change",
    "as the <strong>largest</strong> threat to humandkind",
  ];
  let container;

  let index;
  let activeIndex;
  $: if (container) {
    let scroller = scrollama();
    scroller
      .setup({
        step: ".warning",
        offset: 0.8,
        progress: false,
      })
      .onStepEnter((e) => {
        activeIndex = e.index;
      });
  }
</script>

<div class="container" bind:this={container}>
  {#each warnings as warn, i}
    <div class="warning">
      <div
        in:fade
        class="warning-text"
        style="transition: all 1s;"
        style:opacity={i === activeIndex ? 1 : 0.1}
      >
        {@html warn}
      </div>
    </div>
  {/each}
</div>

<style lang="scss">
  .warning {
    position: sticky;
    width: 100vw;
    top: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 3rem;
    background-image: linear-gradient(black, black);
    background-size: cover;
    background-repeat: no-repeat;
    height: 400px;
    padding: 200px 0;
    background-attachment: fixed;

    & .warning-text {
      position: absolute;
      color: white;
    }
  }

  div.warning:nth-child(5) {
    padding-top: 0;
    position: relative;
  }

  .warning:first-child {
    padding-top: 300px;
  }
</style>
