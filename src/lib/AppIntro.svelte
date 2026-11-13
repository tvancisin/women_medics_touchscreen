<script lang="ts">
  export let onExplore: () => void = () => {};
  export let onHome: () => void = () => {};
</script>

<section class="intro-screen" aria-labelledby="intro-title">
  <a class="wordmark" href="#experience" on:click|preventDefault={onHome}>
    Women Medics
  </a>

  <div class="intro-content">
    <p class="eyebrow">Edinburgh, 1582–2026</p>
    <h1 id="intro-title">Women’s medical education in Edinburgh</h1>
    <p class="description">
      Follow the people, places and turning points that reshaped who could study medicine.
    </p>
    <button class="explore-button" type="button" on:click={onExplore}>
      Start exploring
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
    background: linear-gradient(
      180deg,
      rgba(8, 8, 8, 0.56) 0%,
      rgba(8, 8, 8, 0.08) 44%,
      rgba(8, 8, 8, 0.78) 100%
    );
    color: #fff;
    pointer-events: none;
  }

  .wordmark,
  .intro-content {
    pointer-events: auto;
  }

  .wordmark {
    width: fit-content;
    color: #fff;
    font-size: clamp(1rem, 3vw, 1.375rem);
    font-weight: 700;
    letter-spacing: normal;
    line-height: 1;
    text-decoration: none;
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
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }

  h1 {
    max-width: 10ch;
    margin: 0;
    font-size: clamp(2.45rem, 10vw, 6.75rem);
    font-weight: 650;
    letter-spacing: -0.05em;
    line-height: 0.9;
  }

  .description {
    max-width: 35rem;
    margin: 1.25rem 0 0;
    font-size: clamp(1rem, 3.5vw, 1.2rem);
    line-height: 1.45;
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

  .wordmark:focus-visible,
  .explore-button:focus-visible {
    outline: 3px solid #fff;
    outline-offset: 4px;
  }
</style>
