<script lang="ts">
  import * as d3 from "d3";
  import { onDestroy } from "svelte";

  type WomenMedicsDatum = { year: number; number: number };

  export let width: number;
  export let currentYear: number;
  export let domainStartYear: number;
  export let domainEndYear: number;
  export let timelineY: number;
  export let yearToX: (year: number) => number;
  export let womenMedicsData: WomenMedicsDatum[] = [];
  export let menMedicsData: WomenMedicsDatum[] = [];
  export let menComparisonActive = false;
  export let showWomenMedicsMarker = false;
  // Share the actual chart position with the HTML note anchored above it.
  export let markerY = 0;

  const chartPaddingBottom = 0;
  const chartPaddingTop = 50;
  const menMissingDataStartYear = 1833;
  const trailingMissingDataEndYear = 1970;
  const menComparisonStartYear = 1966;
  const menChartStartYear = 1836;
  const menChartEndYear = 1966;
  const menChartYearsPerSecond = 10;
  let menDrawYear = menChartStartYear;
  let menAnimationFrameId: number | null = null;
  let hasStartedMenComparison = false;

  const resetMenComparison = () => {
    if (menAnimationFrameId !== null) cancelAnimationFrame(menAnimationFrameId);
    menAnimationFrameId = null;
    hasStartedMenComparison = false;
  };

  const startMenComparison = () => {
    hasStartedMenComparison = true;
    menDrawYear = menChartStartYear;
    const startedAt = performance.now();
    const step = (now: number) => {
      menDrawYear = Math.min(
        menChartEndYear,
        menChartStartYear + ((now - startedAt) / 1_000) * menChartYearsPerSecond,
      );
      menAnimationFrameId =
        menDrawYear < menChartEndYear ? requestAnimationFrame(step) : null;
    };
    menAnimationFrameId = requestAnimationFrame(step);
  };

  $: showMenComparison =
    menComparisonActive && currentYear >= menComparisonStartYear;
  // Like the main timeline, this data sequence has its own animated year.
  // It runs through the complete men's series while the main year stays 1966.
  $: if (menComparisonActive && menMedicsData.length && !hasStartedMenComparison) {
    startMenComparison();
  } else if (!menComparisonActive && hasStartedMenComparison) {
    resetMenComparison();
  }
  onDestroy(resetMenComparison);

  export const isSettled = () =>
    !menComparisonActive || !menMedicsData.length || menDrawYear === menChartEndYear;

  $: projectionData =
    womenMedicsData.length === 0
      ? []
      : (() => {
          const last = [...womenMedicsData]
            .sort((a, b) => a.year - b.year)
            .at(-1);

          if (!last) return [];

          const data = [{ year: last.year, number: last.number }];

          for (let year = last.year + 1; year <= 2026; year++) {
            data.push({
              year,
              number: last.number + (year - last.year) * 2,
            });
          }

          return data;
        })();

  $: projectionVisibleData =
    currentYear <= 1966
      ? []
      : projectionData.filter(
          (d) =>
            d.year <= currentYear &&
            d.year >= domainStartYear &&
            d.year <= domainEndYear,
        );

  $: projectionPath = d3
    .line<WomenMedicsDatum>()
    .x((d) => yearToX(d.year))
    .y((d) => yScale(d.number))(projectionVisibleData);

  // Keep values up to the current animated year only.
  $: visibleData = womenMedicsData
    .filter(
      (d) =>
        d.year <= currentYear &&
        d.year >= domainStartYear &&
        d.year <= domainEndYear,
    )
    .sort((a, b) => a.year - b.year);

  $: sortedData = [...womenMedicsData].sort((a, b) => a.year - b.year);
  $: sortedMenData = [...menMedicsData].sort((a, b) => a.year - b.year);

  // Both charts use one fixed scale so their values remain directly comparable.
  $: maxY =
    d3.max(
      [...womenMedicsData, ...menMedicsData],
      (d: WomenMedicsDatum) => d.number,
    ) ?? 0;

  // SVG y grows downward, so the range is inverted: bigger values map higher up.
  $: yScale = d3
    .scaleLinear()
    .domain([0, maxY > 0 ? maxY : 1])
    .range([timelineY - chartPaddingBottom, chartPaddingTop]);

  // Show the comparison scale with the men's series. Its horizontal tick
  // lines double as a full-width grid.
  $: showYAxis = showMenComparison && maxY > 0;
  // D3 orders ticks from the bottom value to the top value. Remove that
  // final tick so the y-axis does not have a tick at its upper endpoint.
  $: yTickValues = showYAxis
    ? yScale.ticks(4).slice(0, -1)
    : [];
  $: yTickLabel = d3.format(",");

  $: areaPath = d3
    .area<WomenMedicsDatum>()
    .x((d: WomenMedicsDatum) => yearToX(d.year))
    .y0(timelineY - chartPaddingBottom)
    .y1((d: WomenMedicsDatum) => yScale(d.number))(visibleData);

  $: linePath = d3
    .line<WomenMedicsDatum>()
    .x((d: WomenMedicsDatum) => yearToX(d.year))
    .y((d: WomenMedicsDatum) => yScale(d.number))(visibleData);

  $: menCurrentValue = interpolateValueAtYear(sortedMenData, menDrawYear);
  $: visibleMenData = (() => {
    if (!showMenComparison) return [];
    const visible = sortedMenData.filter(
      (d) =>
        d.year <= menDrawYear &&
        d.year >= domainStartYear &&
        d.year <= domainEndYear,
    );
    // Include the moving tip so the area, line and circle stay attached
    // between recorded years, without changing the underlying data.
    const previous = visible.at(-1);
    if (
      previous &&
      menDrawYear > previous.year &&
      menCurrentValue !== null &&
      menDrawYear >= domainStartYear &&
      menDrawYear <= domainEndYear
    ) {
      visible.push({ year: menDrawYear, number: menCurrentValue });
    }
    return visible;
  })();

  // Continue the first observed year-to-year change backwards to signal the
  // period before the men's series begins. This is a visual cue only, not
  // additional data.
  $: menMissingnessData = (() => {
    const sortedMenData = [...menMedicsData].sort((a, b) => a.year - b.year);
    const [first, second] = sortedMenData;

    if (!first || !second || first.year <= menMissingDataStartYear) return [];

    const yearlyChange =
      (second.number - first.number) / (second.year - first.year);

    return [
      {
        year: menMissingDataStartYear,
        number:
          first.number + (menMissingDataStartYear - first.year) * yearlyChange,
      },
      first,
    ];
  })();

  $: menMissingnessPath = d3
    .line<WomenMedicsDatum>()
    .x((d) => yearToX(d.year))
    .y((d) => yScale(d.number))(menMissingnessData);

  $: showMenMissingness =
    showMenComparison &&
    menMissingnessData.length === 2 &&
    menMissingnessPath;

  // Continue the final observed direction for a short distance beyond the
  // men's data to signal where the series becomes unavailable.
  $: menTrailingMissingnessData = (() => {
    const sortedMenData = [...menMedicsData].sort((a, b) => a.year - b.year);
    const last = sortedMenData.at(-1);
    const previous = sortedMenData.at(-2);

    if (!last || !previous || last.year >= trailingMissingDataEndYear) return [];

    const yearlyChange =
      (last.number - previous.number) / (last.year - previous.year);

    return [
      last,
      {
        year: trailingMissingDataEndYear,
        number:
          last.number +
          (trailingMissingDataEndYear - last.year) * yearlyChange,
      },
    ];
  })();

  $: menTrailingMissingnessPath = d3
    .line<WomenMedicsDatum>()
    .x((d) => yearToX(d.year))
    .y((d) => yScale(d.number))(menTrailingMissingnessData);

  $: showMenTrailingMissingness =
    showMenComparison &&
    menDrawYear >= (menTrailingMissingnessData[0]?.year ?? Infinity) &&
    menTrailingMissingnessData.length === 2 &&
    menTrailingMissingnessPath;

  $: menAreaPath = d3
    .area<WomenMedicsDatum>()
    .x((d: WomenMedicsDatum) => yearToX(d.year))
    .y0(timelineY - chartPaddingBottom)
    .y1((d: WomenMedicsDatum) => yScale(d.number))(visibleMenData);

  $: menLinePath = d3
    .line<WomenMedicsDatum>()
    .x((d: WomenMedicsDatum) => yearToX(d.year))
    .y((d: WomenMedicsDatum) => yScale(d.number))(visibleMenData);

  const interpolateValueAtYear = (
    data: WomenMedicsDatum[],
    year: number,
  ): number | null => {
    if (data.length === 0) {
      return null;
    }

    if (year <= data[0].year) {
      return data[0].number;
    }

    for (let i = 1; i < data.length; i += 1) {
      const previous = data[i - 1];
      const next = data[i];

      if (year <= next.year) {
        const yearSpan = next.year - previous.year;
        if (yearSpan <= 0) {
          return next.number;
        }

        const t = (year - previous.year) / yearSpan;
        return previous.number + (next.number - previous.number) * t;
      }
    }

    return data[data.length - 1].number;
  };

  $: currentValue = interpolateValueAtYear(sortedData, currentYear);
  $: markerX = yearToX(currentYear);
  $: markerY = currentValue === null ? timelineY : yScale(currentValue);
  $: markerLabel =
    currentValue === null
      ? ""
      : `${Math.round(currentValue)} women medics`;
  $: currentYearIsVisible =
    currentYear >= domainStartYear && currentYear <= domainEndYear;
</script>

{#if showMenMissingness}
  <defs>
    <linearGradient
      id="men-missingness-gradient"
      gradientUnits="userSpaceOnUse"
      x1={yearToX(menMissingnessData[0].year)}
      y1={yScale(menMissingnessData[0].number)}
      x2={yearToX(menMissingnessData[1].year)}
      y2={yScale(menMissingnessData[1].number)}
    >
      <stop offset="0%" stop-color="#7dd3fc" stop-opacity="0" />
      <stop offset="100%" stop-color="#7dd3fc" stop-opacity="1" />
    </linearGradient>
  </defs>
{/if}

{#if showMenTrailingMissingness}
  <defs>
    <linearGradient
      id="men-trailing-missingness-gradient"
      gradientUnits="userSpaceOnUse"
      x1={yearToX(menTrailingMissingnessData[0].year)}
      y1={yScale(menTrailingMissingnessData[0].number)}
      x2={yearToX(menTrailingMissingnessData[1].year)}
      y2={yScale(menTrailingMissingnessData[1].number)}
    >
      <stop offset="0%" stop-color="#7dd3fc" stop-opacity="1" />
      <stop offset="100%" stop-color="#7dd3fc" stop-opacity="0" />
    </linearGradient>
  </defs>
{/if}

{#if showYAxis}
  <g class="y-axis" aria-label="Number of medical students">
    <line
      class="y-axis-line"
      x1="0"
      y1={chartPaddingTop}
      x2="0"
      y2={timelineY}
    />
    {#each yTickValues as tick}
      <g class="y-axis-tick" transform={`translate(0, ${yScale(tick)})`}>
        <line x1="0" y1="0" x2={width} y2="0" />
        <text x="8" y="-6">{yTickLabel(tick)}</text>
      </g>
    {/each}
  </g>
{/if}

{#if currentYear >= 1914 && visibleData.length > 1 && areaPath}
  <path d={areaPath} class="area-shape" />
  {#if linePath}
    <path d={linePath} class="area-top-line" fill="none" />
  {/if}
  {#if showWomenMedicsMarker && currentValue !== null && currentYearIsVisible}
    <text class="line-head-label" x={markerX + 8} y={markerY + 3}>
      {markerLabel}
    </text>
  {/if}
  <!-- {#if projectionPath}
    <path d={projectionPath} class="projection-line" fill="none" />
  {/if} -->
{/if}

{#if showMenMissingness}
  <path
    d={menMissingnessPath}
    class="men-missingness-line"
    fill="none"
    aria-label="Estimated trend before men's student data begins"
  />
  <text
    class="men-missingness-label"
    x={yearToX(menMissingnessData[0].year) - 6}
    y={yScale(menMissingnessData[0].number) - 6}
    text-anchor="end"
  >no data</text>
{/if}

{#if showMenTrailingMissingness}
  <path
    d={menTrailingMissingnessPath}
    class="men-missingness-line"
    fill="none"
    aria-label="Estimated trend after men's student data ends"
  />
  <text
    class="men-missingness-label"
    x={yearToX(menTrailingMissingnessData[1].year) + 6}
    y={yScale(menTrailingMissingnessData[1].number) - 6}
  >no data</text>
{/if}

{#if showMenComparison && visibleMenData.length > 1 && menAreaPath}
  <path d={menAreaPath} class="men-area-shape" />
  {#if menLinePath}
    <path d={menLinePath} class="men-area-top-line" fill="none" />
  {/if}
  {#if menDrawYear === menChartEndYear}
    <text class="area-chart-label men-area-chart-label" x={yearToX(1900)} y={yScale(750)}>
      Men medics
    </text>
    <text class="area-chart-label women-area-chart-label" x={yearToX(1950)} y={yScale(100)}>
      Women medics
    </text>
  {/if}
{/if}

{#if showMenComparison && menCurrentValue !== null && currentYear <= menChartEndYear && menDrawYear >= domainStartYear && menDrawYear <= domainEndYear}
  <circle
    class="line-head men-line-head"
    cx={yearToX(menDrawYear)}
    cy={yScale(menCurrentValue)}
    r="4"
  />
  <text
    class="line-head-label men-line-head-label"
    x={yearToX(menDrawYear) + 8}
    y={yScale(menCurrentValue) + 3}
  >{Math.floor(menDrawYear)} · {Math.round(menCurrentValue)} men medics</text>
{/if}

<style>
  .y-axis-line {
    stroke: rgba(255, 255, 255, 0.1);
    stroke-width: 1;
  }

  .y-axis-tick line {
    stroke: rgba(255, 255, 255, 0.15);
    stroke-width: 1;
    stroke-dasharray: 5 4;
  }

  .y-axis-tick text {
    fill: rgba(255, 255, 255, 0.85);
    font-size: 11px;
  }

  .area-shape {
    fill: rgba(255, 255, 255, 0.1);
    stroke: none;
  }

  .area-top-line {
    stroke: rgba(255, 255, 255, 0.7);
    stroke-width: 1;
  }

  .men-area-shape {
    fill: rgba(255, 255, 255, 0.1);
    stroke: none;
  }

  .men-area-top-line {
    stroke: rgba(255, 255, 255, 0.7);
    stroke-width: 1;
  }

  .area-chart-label {
    font-size: 12px;
    pointer-events: none;
    text-anchor: middle;
  }

  .men-area-chart-label {
    fill: rgba(255, 255, 255, 0.85);
  }

  .women-area-chart-label {
    fill: rgba(255, 255, 255, 0.85);
  }

  .men-missingness-line {
    stroke-width: 1;
  }

  .men-missingness-line[aria-label="Estimated trend before men's student data begins"] {
    stroke: url(#men-missingness-gradient);
  }

  .men-missingness-line[aria-label="Estimated trend after men's student data ends"] {
    stroke: url(#men-trailing-missingness-gradient);
  }

  .men-missingness-label {
    fill: rgba(125, 211, 252, 0.7);
    font-size: 11px;
    pointer-events: none;
  }

  .line-head {
    fill: #fff;
  }

  .line-head-label {
    fill: #fff;
    font-size: 11px;
    pointer-events: none;
  }
  .projection-line {
    stroke: rgba(255, 255, 255, 0.7);
    stroke-width: 1;
    stroke-dasharray: 5 4;
    fill: none;
  }
</style>
