<script lang="ts">
  export let progress = 0;
  export let status = "Preparing the experience";
  export let ready = false;
  export let error = "";
  export let onEnter: () => void = () => {};
  export let onRetry: () => void = () => {};

  $: boundedProgress = Math.max(0, Math.min(100, Math.round(progress)));
</script>

<section class="loading-screen" aria-labelledby="loading-title">
  <div class="loading-content">
    <p class="eyebrow">Women Medics</p>
    <h1 id="loading-title">
      {#if error}
        We could not prepare the experience
      {:else if ready}
        Ready to begin
      {:else}
        Preparing the experience
      {/if}
    </h1>

    {#if error}
      <p class="message" role="alert">{error}</p>
      <button class="action-button" type="button" on:click={onRetry}>
        Try again
      </button>
    {:else}
      <p class="message" role="status" aria-live="polite">{status}</p>
      <div
        class="progress-track"
        role="progressbar"
        aria-label="Experience readiness"
        aria-valuemin="0"
        aria-valuemax="100"
        aria-valuenow={boundedProgress}
        aria-valuetext={`${boundedProgress}% — ${status}`}
      >
        <span class="progress-fill" style:width={`${boundedProgress}%`}></span>
      </div>

      {#if ready}
        <button class="action-button" type="button" on:click={onEnter}>
          Enter
        </button>
      {/if}
    {/if}
  </div>
</section>

<style>
  .loading-screen {
    position: fixed;
    z-index: 30;
    inset: 0;
    display: grid;
    place-items: center;
    padding: clamp(1.25rem, 5vw, 4rem);
    background: #f5f0e6;
    color: #151c24;
  }

  .loading-content {
    width: min(100%, 32rem);
  }

  .eyebrow {
    margin: 0 0 0.75rem;
    color: #495563;
    font-size: 0.75rem;
    font-weight: 700;
    letter-spacing: 0.12em;
    text-transform: uppercase;
  }

  h1 {
    max-width: 12ch;
    margin: 0;
    font-size: clamp(2.25rem, 9vw, 5.5rem);
    font-weight: 650;
    letter-spacing: -0.05em;
    line-height: 0.9;
  }

  .message {
    min-height: 1.5em;
    margin: 1.5rem 0 0.75rem;
    color: #151c24;
    font-size: clamp(1rem, 2.75vw, 1.125rem);
    font-weight: 700;
    letter-spacing: -0.05em;
    line-height: 1.45;
  }

  .progress-track {
    width: 100%;
    height: 0.35rem;
    overflow: hidden;
    border-radius: 999px;
    background: #d4d0c9;
  }

  .progress-fill {
    display: block;
    height: 100%;
    border-radius: inherit;
    background: #495563;
    transition: width 180ms ease-out;
  }

  .action-button {
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

  .action-button:hover {
    filter: brightness(0.9);
  }

  .action-button:focus-visible {
    outline: 3px solid #151c24;
    outline-offset: 3px;
  }

  @media (prefers-reduced-motion: reduce) {
    .progress-fill {
      transition: none;
    }
  }
</style>
