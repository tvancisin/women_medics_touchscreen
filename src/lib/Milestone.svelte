<script lang="ts">
  import { tick } from "svelte";
  import { cubicOut } from "svelte/easing";
  import { tweened } from "svelte/motion";

  export let x = 0;
  export let labelX: number | undefined = undefined;
  export let height = 0;
  export let label = "";
  export let active = false;
  export let activeMilestoneStemTipBottomOffset = 90;
  export let mutedInactiveLabel = false;
  export let expandedInactive = false;
  export let expandedPathLength = 250;
  export let activePathLength: number | undefined = undefined;

  const collapsedPathLength = 20;
  const inactiveLabelMaxLength = 18;
  $: shortenedLabel =
    expandedInactive || label.length <= inactiveLabelMaxLength
      ? label
      : `${label.slice(0, inactiveLabelMaxLength).trimEnd()}…`;
  $: labelPaddingX = active || expandedInactive ? 4 : 3;
  $: labelBackgroundHeight = active || expandedInactive ? 20 : 14;
  $: labelBackgroundOffsetX = active || expandedInactive ? 16 : 8;
  $: labelTextOffsetX = active || expandedInactive ? 20 : 11;
  $: labelBackgroundCenterOffsetY = active || expandedInactive ? 3 : 1;
  let labelTextElement: SVGTextElement | null = null;
  let labelWidth = 0;
  $: labelBackgroundWidth = Math.ceil(labelWidth) + labelPaddingX * 2;

  // The stem tip and label share one tween so crowded labels cannot snap
  // sideways independently when an active card collapses.
  const stemTip = tweened({ x, length: 0 }, {
    duration: 500,
    easing: cubicOut,
  });
  $: startY = height - 30;
  $: middleY = height - activeMilestoneStemTipBottomOffset;
  $: fullLength = Math.max(0, startY - middleY);
  $: inactivePathLength = expandedInactive
    ? expandedPathLength
    : Math.min(fullLength, collapsedPathLength);
  $: targetPathLength = active
    ? (activePathLength ?? fullLength)
    : inactivePathLength;

  // Animate the first target too: a newly reached milestone grows upward
  // from zero length at its year instead of appearing at full height.
  $: stemTip.set({
    x: active ? x : (labelX ?? x),
    length: targetPathLength,
  });

  export const isSettled = () =>
    $stemTip.x === (active ? x : (labelX ?? x)) &&
    $stemTip.length === targetPathLength;

  $: endY = startY - $stemTip.length;
  $: labelTextAnchorX = $stemTip.x;
  $: labelRotationX = labelTextAnchorX + labelBackgroundCenterOffsetY;
  $: bendSegmentLength = Math.min(6, Math.max(0, (startY - endY) / 3));
  $: bendStartY = startY - bendSegmentLength;
  $: bendEndY = endY + bendSegmentLength;
  $: bendControlOffset = Math.min(4, bendSegmentLength);
  // This same curve becomes straight when the tip is above its year anchor.
  // Keeping one geometry avoids an abrupt path switch on deactivation.
  $: pathD = `M ${x} ${startY} L ${x} ${bendStartY} C ${x} ${bendStartY - bendControlOffset}, ${labelTextAnchorX} ${bendEndY + bendControlOffset}, ${labelTextAnchorX} ${bendEndY} L ${labelTextAnchorX} ${endY}`;

  async function updateLabelWidth() {
    await tick();

    if (!labelTextElement) {
      labelWidth = 0;
      return;
    }

    labelWidth = labelTextElement.getComputedTextLength();
  }

  // Inactive labels use shortened text and a smaller font, so remeasure them.
  $: if (shortenedLabel && !active) {
    updateLabelWidth();
  }
</script>

<path
  class="event-path"
  class:inactive={!active}
  class:muted-inactive={!active && mutedInactiveLabel}
  d={pathD}
  fill="none"
/>
{#if label && !active}
  <g
    transform="rotate(-90, {labelRotationX}, {endY})"
  >
    <rect
      class="event-label-background"
      x={labelRotationX + labelBackgroundOffsetX}
      y={endY - labelBackgroundCenterOffsetY - labelBackgroundHeight / 2}
      width={labelBackgroundWidth}
      height={labelBackgroundHeight}
      rx="2"
    />
    <text
      bind:this={labelTextElement}
      class="event-label"
      class:inactive={!active}
      class:muted-inactive={mutedInactiveLabel}
      class:expanded-inactive={expandedInactive}
      x={labelRotationX + labelTextOffsetX}
      y={endY + 2}
      text-anchor="start"
    >
      {shortenedLabel}
    </text>
  </g>
{/if}

<style>
  .event-path {
    stroke: rgb(255, 255, 255);
    stroke-width: 2px;
    stroke-linecap: round;
    pointer-events: none;
  }

  .event-path.inactive {
    stroke: #ffffff;
  }

  .event-path.inactive.muted-inactive {
    stroke: rgba(180, 180, 180, 0.95);
  }

  .event-label-background {
    fill: #000;
    pointer-events: none;
  }

  .event-label {
    fill: #fafafa;
    font-size: 16px;
    font-weight: 500;
    pointer-events: none;
  }

  .event-label.inactive {
    fill: #ffffff;
    font-size: 10px;
  }

  .event-label.inactive.muted-inactive {
    fill: rgba(180, 180, 180, 0.95);
  }

  .event-label.inactive.expanded-inactive {
    font-size: 16px;
  }
</style>
