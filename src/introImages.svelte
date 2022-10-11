<script>
  import { fade, fly } from "svelte/transition";

  export let imgtext;
  export let index;
  export let mobile;
</script>

<div class="opening-images">
  <div
    class="introtext"
    style:position={index === 0 ? "fixed" : "sticky"}
    style:font-size={mobile ? "2rem" : "3rem"}
  >
    {#each imgtext as text, i}
      <span in:fade out:fade>
        {@html text}
      </span>
    {/each}
  </div>
  <div class="introimg image1" />
  <div class="introimg image2" />
  <div class="introimg image3" />
  <div class="introimg image4" />
</div>

<style lang="scss">
  .opening-images {
    position: relative;

    .introtext {
      top: 0;
      color: white;
      width: fit-content;
      transition: all 1s;
    }

    & .image1 {
      height: 100vh;
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
      background-attachment: fixed;
      background-image: url("/images/mine1.png");
    }

    & .image2 {
      height: 100vh;
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
      background-attachment: fixed;
      background-image: url("/images/mine2.png");
    }

    & .image3 {
      height: 100vh;
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
      background-attachment: fixed;
      background-image: url("/images/mine3.png");
    }

    & .image4 {
      height: 100vh;
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
      // background-attachment: fixed;
      background-image: url("/images/mine4.png");
    }
  }
</style>
