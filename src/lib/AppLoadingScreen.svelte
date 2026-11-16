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
        error
      {:else if ready}
        ready
      {:else}
        loading
      {/if}
    </h1>

    {#if error}
      <p class="message" role="alert">{error}</p>
      <button class="action-button" type="button" on:click={onRetry}>
        Try again
      </button>
    {:else}
      <p class="sr-status" role="status" aria-live="polite">{status}</p>
      <!-- <p class="message message--placeholder" aria-hidden="true">&nbsp;</p> -->
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

      <button
        class="action-button"
        class:action-button--hidden={!ready}
        type="button"
        aria-hidden={!ready}
        tabindex={ready ? 0 : -1}
        on:click={onEnter}
      >
        Enter
      </button>
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
    background: #080808;
    color: #fff;
  }

  .loading-content {
    width: min(100%, 32rem);
  }

  .eyebrow {
    margin: 0 0 0.1rem;
    color: #fff;
    font-size: 0.75rem;
    font-weight: 700;
    text-transform: uppercase;
  }

  h1 {
    max-width: 12ch;
    margin: 0;
    font-size: clamp(2.25rem, 9vw, 2rem);
    font-weight: 650;
    line-height: 0.9;
  }

  .message {
    min-height: 1.5em;
    margin: 1.5rem 0 0.75rem;
    color: #fff;
    font-size: clamp(1rem, 2.75vw, 1.125rem);
    font-weight: 700;
    line-height: 1.45;
  }

  .sr-status {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }

  .progress-track {
    width: 100%;
    height: 0.35rem;
    overflow: hidden;
    border-radius: 999px;
    background: #303b46;
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
    margin-top: 1rem;
    padding: 0.2rem 0.9rem;
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

  .action-button--hidden {
    visibility: hidden;
  }

  .action-button:focus-visible {
    outline: 3px solid #fff;
    outline-offset: 3px;
  }

  @media (prefers-reduced-motion: reduce) {
    .progress-fill {
      transition: none;
    }
  }
</style>
