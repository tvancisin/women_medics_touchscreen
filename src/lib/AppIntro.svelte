<script lang="ts">
  export let onExplore: () => void = () => {};

  const baseUrl = import.meta.env.BASE_URL;
  const publicUrl = (path: string) => `${baseUrl}${path}`;
</script>

<section class="intro-screen" aria-labelledby="intro-title">
  <img
    class="efi-logo"
    src={publicUrl("img/efi_logo_white.svg")}
    alt="Edinburgh Futures Institute"
  />

  <div class="intro-content">
    <p class="eyebrow">Edinburgh, 1582–2026</p>
    <h1 id="intro-title">Women’s medical education in Edinburgh</h1>
    <button class="explore-button" type="button" on:click={onExplore}>
      Start
    </button>
  </div>
</section>

<style>
  .intro-screen {
    position: fixed;
    z-index: 20;
    inset: 0;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: clamp(1rem, 4vw, 2.5rem);
    background: black;
    color: #fff;
    pointer-events: none;
  }

  .intro-content {
    pointer-events: auto;
  }

  .efi-logo {
    position: absolute;
    top: 2px;
    left: 2px;
    width: auto;
    height: 60px;
    object-fit: contain;
  }

  .intro-content {
    max-width: min(100%, 42rem);
    margin-top: auto;
  }

  .eyebrow {
    margin: 0 0 0.7rem;
    color: #fff;
    font-size: 0.75rem;
    font-weight: 700;
    text-transform: uppercase;
  }

  h1 {
    max-width: 10ch;
    margin: 0;
    font-size: clamp(2rem, 8vw, 5rem);
    font-weight: 650;
    line-height: 0.9;
  }

  .explore-button {
    min-height: 2.75rem;
    margin-top: 1.5rem;
    padding: 0.65rem 1.1rem;
    border: 0;
    border-radius: 0.2rem;
    background: #495563;
    color: #fff;
    font-weight: 700;
    line-height: 1.2;
    cursor: pointer;
  }

  .explore-button:hover {
    filter: brightness(1.14);
  }

  .explore-button:focus-visible {
    outline: 3px solid #fff;
    outline-offset: 4px;
  }
</style>
