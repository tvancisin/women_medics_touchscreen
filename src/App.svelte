<script lang="ts">
  import { onMount } from "svelte";
  import { cubicOut } from "svelte/easing";
  import { tweened } from "svelte/motion";
  import { getCSV, getJson } from "./lib/utils/loaders";
  import { creditsByYear } from "./lib/utils/const";
  import AppIntro from "./lib/AppIntro.svelte";
  import AppLoadingScreen from "./lib/AppLoadingScreen.svelte";
  import BackgroundMap from "./lib/BackgroundMap.svelte";
  import MainTimeline from "./lib/MainTimeline.svelte";

  const baseUrl = import.meta.env.BASE_URL;
  const publicUrl = (path: string) => `${baseUrl}${path}`;

  const startYear = 1582;
  const universityEstablishedYear = 1582;
  const endYear = 2026;

  const timelineZoomTriggerYear = 1862;
  const timelineResetTriggerYear = 1914;
  const timelineZoomDomainStart = 1850;
  const timelineZoomDomainEnd = 1915;
  const timelineZoomDurationMs = 1600;
  const timelineStretchTriggerYear = 1886;
  const timelineStretchStartYear = 1886;
  const timelineStretchEndYear = 1911;
  const timelineStretchResetYear = 1912;
  const timelineStretchTargetStartYear = 1850;
  const timelineStretchTargetEndYear = 1950;
  // Set to false to keep the timeline at its full 1550–2026 range.
  const enableTimelineSpreading = false;
  const timelineAdvanceYearsPerSecond = 50;
  const minimumTimelineAdvanceDurationMs = 600;
  const maximumTimelineAdvanceDurationMs = 3_000;
  const margin = { top: 20, right: 40, bottom: 30, left: 40 };

  // keeping the detail div inside screen
  const milestoneCardWidth = 400;
  const universityFoundedImageAspectRatio = 7272 / 5461;
  const milestoneCardBottomOffset = 65;
  const activeMilestoneStemTipBottomOffset = 60;
  const milestoneCardStemGap =
    milestoneCardBottomOffset - activeMilestoneStemTipBottomOffset;
  const equalMedicalEducationCardBottomOffset = tweened(
    milestoneCardBottomOffset,
    { duration: 500, easing: cubicOut },
  );
  const milestoneCardTopClearance = 65;
  // Heading (22px), its vertical margins (13px), and the card's 5px top/bottom padding.
  const universityFoundedCardChromeHeight = 45;
  // The 1583 map credit occupies one line plus its vertical margins.
  const universityFoundedMapCreditHeight = 28;

  // Anchor cards differently depending on whether the milestone is left or right of center.
  const card_left = [1726, 1809];
  const card_middle = [1911];
  const clampedLeft = (x: number, year: number, timelineYear = year) => {
    // return x < width / 2 ? x + 10 : x - milestoneCardWidth - 10;
    if (card_middle.includes(timelineYear)) {
      const cardWidth =
        year === 1911
          ? Math.min(500, Math.max(0, width - 32))
          : milestoneCardWidth;
      return x - cardWidth / 2;
    }
    if (card_left.includes(year)) {
      return x - milestoneCardWidth;
    } else {
      return x;
    }
  };

  type Milestone = {
    id: string;
    year: number;
    contentYear: number;
    label: string;
    renderOnTimeline?: boolean;
  };
  const milestones: Milestone[] = [
    {
      id: "university-founded",
      year: 1583,
      contentYear: 1583,
      label: "University Foundation 1582",
    },
    // Pause for the map overlay without drawing a timeline milestone stem.
    {
      id: "map-1682",
      year: 1682,
      contentYear: 1682,
      label: "",
      renderOnTimeline: false,
    },
    {
      id: "school-of-medicine",
      year: 1726,
      contentYear: 1726,
      label: "School of Medicine 1726",
    },
    {
      id: "james-barry",
      year: 1809,
      contentYear: 1809,
      label: "Margaret Bulkley/James Barry 1809",
    },
    {
      id: "elizabeth-garrett",
      year: 1862,
      contentYear: 1862,
      label: "Elizabeth Garrett 1862",
    },
    {
      id: "first-classes",
      year: 1867,
      contentYear: 1867,
      label: "First Classes for Women 1867",
    },
    {
      id: "edinburgh-forty",
      year: 1869,
      contentYear: 1869,
      label: "Edinburgh Seven/Forty 1869",
    },
    {
      id: "surgeons-hall-riot",
      year: 1870,
      contentYear: 1870,
      label: "The Riot 1870",
    },
    {
      id: "physiology-students",
      year: 1875,
      contentYear: 1875,
      label: "Physiology Students 1875",
    },
    {
      id: "school-for-women",
      year: 1886,
      contentYear: 1886,
      label: "School of Medicine for Women 1886",
    },
    {
      id: "college-for-women",
      year: 1889,
      contentYear: 1889,
      label: "College of Medicine for Women 1889",
    },
    {
      id: "school-and-college-students",
      year: 1911,
      contentYear: 1911,
      label: "School and College Students 1911",
    },
    {
      id: "career-locations",
      year: 1911,
      contentYear: 1915,
      label: "Career Locations in 1915",
    },
    {
      id: "equal-medical-education",
      year: 1916,
      contentYear: 1916,
      label: "Equal Medical Education 1916",
    },
    {
      id: "international-female-medics",
      year: 1926,
      contentYear: 1926,
      label: "International female medics 1926",
    },
  ];

  const splitMilestoneYears = new Set([
    1809, 1862, 1867, 1870, 1875, 1886, 1889, 1911,
  ]);
  const universityFoundedMapCredit =
    creditsByYear.find((credits) => credits.year === 1583)?.map ?? "";
  const schoolOfMedicineMapCredit =
    creditsByYear.find((credits) => credits.year === 1726)?.map ?? "";
  const firstClassesMapCredit =
    creditsByYear.find((credits) => credits.year === 1867)?.map ?? "";
  const surgeonsHallRiotMapCredit =
    creditsByYear.find((credits) => credits.year === 1870)?.map ?? "";
  const physiologyStudentsMapCredit =
    creditsByYear.find((credits) => credits.year === 1875)?.map ?? "";
  const schoolForWomenMapCredit =
    creditsByYear.find((credits) => credits.year === 1886)?.map ?? "";
  const collegeForWomenMapCredit =
    creditsByYear.find((credits) => credits.year === 1889)?.map ?? "";
  // const womenDoctorsWarMilestoneYear = 1919;

  let height = 0;
  let width = 0;
  let garrettJourneyData: unknown = null;
  let barryJourneyData: unknown = null;
  let riotData: unknown = null;
  let firstClassesPathsData: unknown = null;
  let physiologyPathsData: unknown = null;
  let colonies: unknown = null;
  let suez: unknown = null;
  let edinburghRoutes: unknown = null;
  let womenDoctorsWarData: unknown = null;
  let womenDoctorsData: unknown = null;
  let womenCareers1915Data: unknown = null;
  let internationalMedics1926Data: Array<Record<string, string>> = [];
  let currentYear = startYear;
  let timelineAdvanceFrameId: number | null = null;
  let isAdvancingTimeline = false;
  let timelineZoomFrameId: number | null = null;
  let timelineStretchFrameId: number | null = null;
  let hasZoomedTimeline = false;
  let hasResetTimeline = false;
  let hasStretchedTimeline = false;
  let hasResetTimelineStretch = false;
  let timelineStretchProgress = 0;
  let timelineDomainStart = startYear;
  let timelineDomainEnd = endYear;
  let activeMilestoneIndex = -1;
  let pausedAtYear: number | null = null;
  let pausedMilestoneId: string | null = null;
  let csvDataReady = false;
  let jsonDataReady = false;
  let fontsReady = false;
  let mapReady = false;
  let mapWarmupProgress = 0;
  let loadError = "";
  let mapInstanceKey = 0;
  let hasDismissedLoading = false;
  let introVisible = false;
  let hasEnteredExperience = false;
  let reloadData: (() => void) | null = null;

  $: readinessComplete = [csvDataReady, jsonDataReady, fontsReady, mapReady];
  $: readinessCount = readinessComplete.filter(Boolean).length;
  $: loadingProgress =
    ((csvDataReady ? 1 : 0) +
      (jsonDataReady ? 1 : 0) +
      (fontsReady ? 1 : 0) +
      mapWarmupProgress) /
    readinessComplete.length *
    100;
  $: isReadyToEnter = readinessCount === readinessComplete.length && !loadError;
  $: loadingStatus = !fontsReady
    ? "Loading type and interface"
    : !csvDataReady || !jsonDataReady
      ? "Loading historical data"
      : !mapReady
        ? mapWarmupProgress > 0
          ? "Warming story map views"
          : "Loading the map"
        : "Ready";

  $: equalMedicalEducationCardBottomOffset.set(
    pausedMilestoneId === "equal-medical-education"
      ? height / 2 + milestoneCardStemGap
      : milestoneCardBottomOffset,
  );
  let womenMedicsData: Array<{ year: number; number: number }> = [];
  let menMedicsData: Array<{ year: number; number: number }> = [];
  $: activeCredits = creditsByYear.find(
    (credits) => credits.year === pausedAtYear,
  );
  $: mapCredit = activeCredits?.map ?? "";
  $: imageCredit = activeCredits?.image ?? "";
  let edinburghSevenData: Array<Record<string, string>> = [];
  const edinburghSevenNames = [
    "Sophia Jex-Blake",
    "Isabel Pryer",
    "Mary Pechey",
    "Matilda Chaplin",
    "Helen Evans",
    "Mary Anderson",
    "Emily Bovell",
  ];
  const edinburghSevenOrder = new Map(
    edinburghSevenNames.map((name, index) => [name, index]),
  );

  type FirstClassesGeoDatum = {
    source_data?: {
      entry_year?: number | string;
    };
  };
  let firstClassesGeoData: FirstClassesGeoDatum[] = [];

  type WomenDoctorDatum = {
    source_data?: {
      "Year of student registration"?: number | string | null;
    };
  };

  type WomenCareer1915Datum = {
    source_data?: {
      career_location_1915?: {
        country?: string | null;
        region?: string | null;
      } | null;
    };
  };

  type CareerCountryCount = {
    country: string;
    count: number;
    percent: number;
  };

  const edinburghFortyImageUrl = (img?: string) => {
    const fileName = String(img ?? "").trim();
    if (!fileName || fileName.toLowerCase() === "null") return undefined;

    const normalizedPath = fileName.startsWith("/")
      ? fileName.slice(1)
      : `img/edin_forty/${fileName}`;
    const src = publicUrl(normalizedPath);
    return `url("${src}")`;
  };

  const universityFoundedBackgroundImage = `url("${publicUrl("img/edinburgh_1583.jpg")}")`;
  type PhysiologyGeoDatum = {
    source_data?: {
      entry_year?: number | string;
    };
  };
  let womenPhysiologyGeoData: PhysiologyGeoDatum[] = [];

  const getStudentRegistrationYear = (doctor: WomenDoctorDatum) => {
    const rawYear = doctor.source_data?.["Year of student registration"];

    if (rawYear === null || rawYear === undefined || rawYear === "") {
      return Number.POSITIVE_INFINITY;
    }

    const year = Number(rawYear);
    return Number.isFinite(year) ? year : Number.POSITIVE_INFINITY;
  };

  const getAcademicYearEnd = (value: string | undefined) => {
    const [startText, endText] = String(value ?? "")
      .trim()
      .split(/[-–—]/)
      .map((year) => year.trim());
    const startYear = Number(startText);
    const endYear = Number(endText ?? startText);

    if (!Number.isFinite(startYear) || !Number.isFinite(endYear)) {
      return Number.NaN;
    }

    if (!endText || endText.length >= startText.length) {
      return endYear;
    }

    const abbreviatedDigits = 10 ** endText.length;
    const endAcademicYear =
      Math.floor(startYear / abbreviatedDigits) * abbreviatedDigits + endYear;

    return endAcademicYear < startYear
      ? endAcademicYear + abbreviatedDigits
      : endAcademicYear;
  };

  const getCareerCountry = (
    careerLocation: NonNullable<
      WomenCareer1915Datum["source_data"]
    >["career_location_1915"],
  ) => {
    const country = String(careerLocation?.country ?? "").trim();
    if (!country || country.toLowerCase() === "null") return null;

    if (country.toLowerCase() === "democratic republic of the congo") {
      return "Congo";
    }

    if (country.toLowerCase() === "myanmar (burma)") {
      return "Myanmar";
    }

    if (country.toLowerCase() !== "united kingdom") return country;

    const region = String(careerLocation?.region ?? "")
      .trim()
      .toLowerCase();
    if (region === "london") return "England";
    if (["england", "scotland", "wales", "ireland"].includes(region)) {
      return `${region[0].toUpperCase()}${region.slice(1)}`;
    }

    return country;
  };

  const buildCareerCountryCounts = (rawData: unknown): CareerCountryCount[] => {
    if (!Array.isArray(rawData)) return [];

    const countsByCountry = new Map<string, number>();

    for (const row of rawData as WomenCareer1915Datum[]) {
      const country = getCareerCountry(row.source_data?.career_location_1915);
      if (!country) continue;
      countsByCountry.set(country, (countsByCountry.get(country) ?? 0) + 1);
    }

    const countries = Array.from(countsByCountry.entries()).sort(
      ([countryA, countA], [countryB, countB]) =>
        countB - countA || countryA.localeCompare(countryB),
    );
    const maxCount = Math.max(...countries.map(([, count]) => count), 1);

    return countries.map(([country, count]) => ({
      country,
      count,
      percent: (count / maxCount) * 100,
    }));
  };

  $: careerCountryCounts = buildCareerCountryCounts(womenCareers1915Data);
  $: orderedEdinburghFortyData = [...edinburghSevenData].sort(
    (personA, personB) =>
      (edinburghSevenOrder.get(personA.name) ?? Number.MAX_SAFE_INTEGER) -
      (edinburghSevenOrder.get(personB.name) ?? Number.MAX_SAFE_INTEGER),
  );

  $: maxSpan = Math.max(0, width - margin.left - margin.right);
  $: timelineY = Math.max(margin.top, height - margin.bottom);

  $: axisStart = margin.left;
  $: axisRight = axisStart + maxSpan;
  $: timelineDomainSpan = Math.max(1, timelineDomainEnd - timelineDomainStart);
  $: universityFoundedAvailableHeight = Math.max(
    0,
    height - milestoneCardBottomOffset - milestoneCardTopClearance,
  );
  $: universityFoundedPreferredWidth =
    Math.max(
      0,
      universityFoundedAvailableHeight -
        universityFoundedCardChromeHeight -
        universityFoundedMapCreditHeight,
    ) * universityFoundedImageAspectRatio;
  $: universityFoundedCardWidth = Math.min(
    universityFoundedPreferredWidth,
    Math.max(0, width - margin.left),
  );
  $: universityFoundedCardHeight = Math.min(
    universityFoundedAvailableHeight,
    Math.max(
      0,
      universityFoundedCardWidth / universityFoundedImageAspectRatio +
        universityFoundedCardChromeHeight +
        universityFoundedMapCreditHeight,
    ),
  );

  const timelineZoomEase = (t: number) =>
    t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2;

  const startTimelineDomainTransition = (
    targetStartYear: number,
    targetEndYear: number,
    onComplete?: () => void,
  ) => {
    if (timelineZoomFrameId !== null) {
      cancelAnimationFrame(timelineZoomFrameId);
    }

    const fromStart = timelineDomainStart;
    const fromEnd = timelineDomainEnd;
    const startedAt = performance.now();

    const step = (now: number) => {
      const progress = Math.min(1, (now - startedAt) / timelineZoomDurationMs);
      const eased = timelineZoomEase(progress);

      timelineDomainStart = fromStart + (targetStartYear - fromStart) * eased;
      timelineDomainEnd = fromEnd + (targetEndYear - fromEnd) * eased;

      if (progress < 1) {
        timelineZoomFrameId = requestAnimationFrame(step);
      } else {
        timelineDomainStart = targetStartYear;
        timelineDomainEnd = targetEndYear;
        timelineZoomFrameId = null;
        onComplete?.();
      }
    };

    timelineZoomFrameId = requestAnimationFrame(step);
  };

  const startTimelineStretch = (targetProgress: number) => {
    if (timelineStretchFrameId !== null) {
      cancelAnimationFrame(timelineStretchFrameId);
    }

    const fromProgress = timelineStretchProgress;
    const startedAt = performance.now();

    const step = (now: number) => {
      const progress = Math.min(1, (now - startedAt) / timelineZoomDurationMs);
      const eased = timelineZoomEase(progress);
      timelineStretchProgress =
        fromProgress + (targetProgress - fromProgress) * eased;

      if (progress < 1) {
        timelineStretchFrameId = requestAnimationFrame(step);
      } else {
        timelineStretchProgress = targetProgress;
        timelineStretchFrameId = null;
      }
    };

    timelineStretchFrameId = requestAnimationFrame(step);
  };

  // calculating x position for a given year
  $: yearToX = (year: number) => {
    const mainTimelineX = (timelineYear: number) => {
      const yearProgress = (timelineYear - startYear) / (endYear - startYear);
      return axisStart + yearProgress * maxSpan;
    };
    const originalX = mainTimelineX(year);
    const stretchedStartX = mainTimelineX(timelineStretchTargetStartYear);
    const stretchedEndX = mainTimelineX(timelineStretchTargetEndYear);

    let stretchedX: number;
    if (year <= timelineStretchStartYear) {
      const progress =
        (year - startYear) / (timelineStretchStartYear - startYear);
      stretchedX = axisStart + progress * (stretchedStartX - axisStart);
    } else if (year <= timelineStretchEndYear) {
      const progress =
        (year - timelineStretchStartYear) /
        (timelineStretchEndYear - timelineStretchStartYear);
      stretchedX =
        stretchedStartX + progress * (stretchedEndX - stretchedStartX);
    } else {
      const progress =
        (year - timelineStretchEndYear) / (endYear - timelineStretchEndYear);
      stretchedX = stretchedEndX + progress * (axisRight - stretchedEndX);
    }

    return originalX + (stretchedX - originalX) * timelineStretchProgress;
  };

  $: if (currentYear >= timelineStretchTriggerYear && !hasStretchedTimeline) {
    hasStretchedTimeline = true;
    startTimelineStretch(1);
  }

  $: if (currentYear >= timelineStretchResetYear && !hasResetTimelineStretch) {
    hasResetTimelineStretch = true;
    startTimelineStretch(0);
  }

  $: if (
    enableTimelineSpreading &&
    currentYear >= timelineZoomTriggerYear &&
    currentYear < timelineResetTriggerYear &&
    !hasZoomedTimeline
  ) {
    hasZoomedTimeline = true;
    startTimelineDomainTransition(
      timelineZoomDomainStart,
      timelineZoomDomainEnd,
    );
  }

  $: if (
    enableTimelineSpreading &&
    currentYear >= timelineResetTriggerYear &&
    !hasResetTimeline
  ) {
    hasResetTimeline = true;
    hasZoomedTimeline = false;
    startTimelineDomainTransition(startYear, endYear);
  }

  const finishAtMilestone = (index: number) => {
    const milestone = milestones[index];
    if (!milestone) return;

    activeMilestoneIndex = index;
    currentYear = milestone.year;
    pausedAtYear = milestone.year;
    pausedMilestoneId = milestone.id;
  };

  const advanceToMilestone = (index: number) => {
    const milestone = milestones[index];
    if (index > milestones.length || isAdvancingTimeline) return;

    if (timelineAdvanceFrameId !== null) {
      cancelAnimationFrame(timelineAdvanceFrameId);
    }

    const fromYear = currentYear;
    const targetYear = milestone?.year ?? endYear;
    const duration = Math.min(
      maximumTimelineAdvanceDurationMs,
      Math.max(
        minimumTimelineAdvanceDurationMs,
        (Math.abs(targetYear - fromYear) / timelineAdvanceYearsPerSecond) *
          1_000,
      ),
    );
    const startedAt = performance.now();

    pausedAtYear = null;
    pausedMilestoneId = null;
    isAdvancingTimeline = true;

    const step = (now: number) => {
      const progress = Math.min(1, (now - startedAt) / duration);
      currentYear = fromYear + (targetYear - fromYear) * cubicOut(progress);

      if (progress < 1) {
        timelineAdvanceFrameId = requestAnimationFrame(step);
        return;
      }

      currentYear = targetYear;
      timelineAdvanceFrameId = null;
      isAdvancingTimeline = false;
      if (milestone) {
        finishAtMilestone(index);
      } else {
        activeMilestoneIndex = milestones.length;
      }
    };

    timelineAdvanceFrameId = requestAnimationFrame(step);
  };

  const handleNextMilestone = () => {
    advanceToMilestone(activeMilestoneIndex + 1);
  };

  const handleLoadingEnter = () => {
    hasDismissedLoading = true;
    introVisible = true;
  };

  const handleExplore = () => {
    introVisible = false;
    hasEnteredExperience = true;
    advanceToMilestone(0);
  };

  const handleIntroHome = () => {
    currentYear = startYear;
    activeMilestoneIndex = -1;
    pausedAtYear = null;
    pausedMilestoneId = null;
    isAdvancingTimeline = false;
    if (timelineAdvanceFrameId !== null) {
      cancelAnimationFrame(timelineAdvanceFrameId);
      timelineAdvanceFrameId = null;
    }
  };

  const handleMapReady = () => {
    mapWarmupProgress = 1;
    mapReady = true;
  };

  const handleMapWarmupProgress = (progress: number) => {
    mapWarmupProgress = Math.max(0, Math.min(1, progress));
  };

  const handleMapLoadError = () => {
    if (!mapReady) {
      loadError = "The map could not be loaded. Check your connection and try again.";
    }
  };

  const handleRetry = () => {
    loadError = "";
    csvDataReady = false;
    jsonDataReady = false;
    mapReady = false;
    mapWarmupProgress = 0;
    mapInstanceKey += 1;
    reloadData?.();
  };

  // init
  onMount(() => {
    const loadCsvData = async () => {
      try {
        const [
          rawWomenMedicsData,
          rawMenMedicsData,
          rawEdinburghSevenData,
          rawInternationalMedics1926Data,
        ] =
          (await getCSV([
            publicUrl("data/women_medics_1914_1966.csv"),
            publicUrl("data/men_medics_1836_1966.csv"),
            publicUrl("data/edinburgh_forty.csv"),
            publicUrl("data/1926_international_medics.csv"),
          ])) as [
            Array<{ year?: string; number?: string }>,
            Array<Record<string, string>>,
            Array<Record<string, string>>,
            Array<Record<string, string>>,
          ];

        womenMedicsData = rawWomenMedicsData
          .map((row: { year?: string; number?: string }) => {
            const year = getAcademicYearEnd(row.year);
            const total = Number(row.number);
            return { year, number: total };
          })
          .filter(
            (row: { year: number; number: number }) =>
              Number.isFinite(row.year) && Number.isFinite(row.number),
          );

        menMedicsData = rawMenMedicsData
          .map((row: { year?: string; number?: string }) => {
            const year = getAcademicYearEnd(row.year);
            const total = Number(row.number);
            return { year, number: total };
          })
          .filter(
            (row: { year: number; number: number }) =>
              Number.isFinite(row.year) && Number.isFinite(row.number),
          );

        edinburghSevenData = rawEdinburghSevenData;
        internationalMedics1926Data = rawInternationalMedics1926Data;
        csvDataReady = true;

        // firstClassesData = first_classes;
      } catch (error: unknown) {
        console.error("Failed to load", error);
        loadError = "The timeline data could not be loaded. Check your connection and try again.";
      }
    };

    const loadJsonData = async () => {
      try {
        const [
          rawWomenPhysiologyGeoData,
          rawGarrettJourneyData,
          rawBarryJourneyData,
          rawFirstClasses,
          rawFirstClassesPaths,
          rawPhysiologyPaths,
          rawWomenDoctors,
          rawWomenDoctors1915,
          rawColonies,
          rawSuez,
          rawEdinburghRoutes,
          rawWomenWarData,
          rawRiotData,
        ] = await getJson([
          publicUrl("data/women_physiology_geo.json"),
          publicUrl("data/geo/garrett_journey.json"),
          publicUrl("data/geo/barry_journey.json"),
          publicUrl("data/first_women_classes_1867_geo.json"),
          publicUrl("data/geo/walking_paths_first_classes.json"),
          publicUrl("data/geo/walking_paths_physiology.json"),
          publicUrl("data/women_doctors_enhanced.json"),
          publicUrl("data/women_careers_1915.json"),
          publicUrl("data/geo/colonies_1885.json"),
          publicUrl("data/geo/suez_routes.json"),
          publicUrl("data/geo/edinburgh_routes.json"),
          publicUrl("data/women_war.json"),
          publicUrl("data/geo/riot.json"),
        ]);

        womenPhysiologyGeoData = Array.isArray(rawWomenPhysiologyGeoData)
          ? (rawWomenPhysiologyGeoData as PhysiologyGeoDatum[])
          : [];

        firstClassesGeoData = Array.isArray(rawFirstClasses)
          ? (rawFirstClasses as FirstClassesGeoDatum[])
          : [];

        garrettJourneyData = rawGarrettJourneyData ?? null;
        barryJourneyData = rawBarryJourneyData ?? null;
        riotData = rawRiotData ?? null;
        firstClassesPathsData = rawFirstClassesPaths ?? null;
        physiologyPathsData = rawPhysiologyPaths ?? null;
        colonies = rawColonies ?? null;
        suez = rawSuez ?? null;
        edinburghRoutes = rawEdinburghRoutes ?? null;
        womenDoctorsWarData = rawWomenWarData ?? null;
        womenDoctorsData = Array.isArray(rawWomenDoctors)
          ? [...(rawWomenDoctors as WomenDoctorDatum[])].sort((a, b) => {
              return (
                getStudentRegistrationYear(a) - getStudentRegistrationYear(b)
              );
            })
          : null;
        womenCareers1915Data = Array.isArray(rawWomenDoctors1915)
          ? rawWomenDoctors1915
          : null;
        jsonDataReady = true;
      } catch (error: unknown) {
        console.error("Failed to load timeline JSON data", error);
        loadError = "The map data could not be loaded. Check your connection and try again.";
      }
    };

    reloadData = () => {
      loadCsvData();
      loadJsonData();
    };
    reloadData();

    const waitForFonts = async () => {
      try {
        await document.fonts?.ready;
      } catch (error: unknown) {
        console.warn("Font loading check failed", error);
      } finally {
        fontsReady = true;
      }
    };
    waitForFonts();

          // Trigger path shrink for the milestone we're leaving — works
    return () => {
      if (timelineAdvanceFrameId !== null) {
        cancelAnimationFrame(timelineAdvanceFrameId);
      }
      if (timelineZoomFrameId !== null) {
        cancelAnimationFrame(timelineZoomFrameId);
      }
      if (timelineStretchFrameId !== null) {
        cancelAnimationFrame(timelineStretchFrameId);
      }
      reloadData = null;
    };
  });
</script>

<main
  id="experience"
  class:experience-active={hasEnteredExperience}
  bind:clientWidth={width}
  bind:clientHeight={height}
  aria-hidden={!hasEnteredExperience}
  inert={!hasEnteredExperience}
>
  <!-- 1919 map inputs temporarily disabled:
  {womenDoctorsWarData}
  showWomenDoctorsWarLocations={pausedAtYear === womenDoctorsWarMilestoneYear}
  -->
  {#key mapInstanceKey}
    <BackgroundMap
      {currentYear}
      {garrettJourneyData}
      {barryJourneyData}
      {riotData}
      {womenPhysiologyGeoData}
      {firstClassesGeoData}
      {firstClassesPathsData}
      {physiologyPathsData}
      {womenDoctorsData}
      {womenCareers1915Data}
      {colonies}
      {suez}
      {edinburghRoutes}
      {edinburghSevenData}
      {internationalMedics1926Data}
      showWomenDoctorBirthplaces={pausedMilestoneId ===
        "school-and-college-students"}
      showWomenDoctorCareerLocations={pausedMilestoneId === "career-locations"}
      showInternationalMedics1926={pausedMilestoneId ===
        "international-female-medics"}
      onReady={handleMapReady}
      onLoadError={handleMapLoadError}
      onWarmupProgress={handleMapWarmupProgress}
    />
  {/key}
  <img
    class="site-logo"
    src={publicUrl("img/efi_logo_white.svg")}
    alt="Edinburgh Futures Institute"
  />
  <h1 class="page-title">Women's Medical Education in Edinburgh</h1>
  <!-- {#if mapCredit || imageCredit}
    <aside class="map-image-credits" aria-label="Map and image credits">
      {#if mapCredit}
        <em>Map Credit: {mapCredit}</em>
      {/if}
      {#if imageCredit}
        <em>Image Credit: {imageCredit}</em>
      {/if}
    </aside>
  {/if} -->
  {#if
    hasEnteredExperience &&
    !isAdvancingTimeline &&
    activeMilestoneIndex >= 0 &&
    activeMilestoneIndex < milestones.length
  }
    <button
      class="resume-button"
      type="button"
      on:click={handleNextMilestone}
    >
      Next
    </button>
  {/if}
  <MainTimeline
    {width}
    {height}
    {currentYear}
    {startYear}
    {universityEstablishedYear}
    {endYear}
    {timelineDomainStart}
    {timelineDomainEnd}
    {timelineY}
    {axisStart}
    {axisRight}
    {milestones}
    {pausedMilestoneId}
    {activeMilestoneStemTipBottomOffset}
    {womenDoctorsData}
    {womenMedicsData}
    {menMedicsData}
    {timelineStretchProgress}
    {yearToX}
  />

  {#if currentYear === 1682}
    <h2 class="timeline-year-heading">
      Edinburgh and Leith in 1682
      <br />
      John Adair's map of Midlothian, from around 1682
    </h2>
  {/if}

  {#each milestones as milestone (milestone.id)}
    {@const year = milestone.contentYear}
    <div
      class="milestone-card"
      class:milestone-card--university-founded={year === 1583}
      class:milestone-card--edinburgh-forty={year === 1869}
      class:milestone-card--women-doctors-1911={year === 1911}
      class:milestone-card--split={splitMilestoneYears.has(year)}
      class:is-active={pausedMilestoneId === milestone.id}
      style:display={milestone.id === "map-1682" ? "none" : undefined}
      style:bottom={`${
        milestone.year === 1916
          ? $equalMedicalEducationCardBottomOffset
          : milestoneCardBottomOffset
      }px`}
      style:left={`${clampedLeft(yearToX(milestone.year), year, milestone.year)}px`}
      style:width={year === 1583 && pausedMilestoneId === milestone.id
        ? `${universityFoundedCardWidth}px`
        : undefined}
      style:height={year === 1583 && pausedMilestoneId === milestone.id
        ? `${universityFoundedCardHeight}px`
        : undefined}
    >
      <h1 class="milestone-card-heading">{milestone.label}</h1>
      {#if year === 1583}
        <div
          class="university-founded-card-image"
          style:background-image={universityFoundedBackgroundImage}
        ></div>
        <p class="milestone-map-credit">{universityFoundedMapCredit}</p>
      {:else if year === 1726}
        <div class="milestone-card-text-only">
          <div class="milestone-card-title">
            In 1726, when the School of Medicine was established, the population
            of Edinburgh was roughly 40,000 people. The education took place at
            the Old College (currently School of Law) as well as at the Old
            Surgeon's Hall.
          </div>
          <div class="milestone-map-section">
            <p class="milestone-map-credit">
              Map: {schoolOfMedicineMapCredit}
            </p>
          </div>
        </div>
      {:else if year === 1809}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/barry.jpg")}
              alt="Margaret Bulkley / James Barry"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              Born as Margaret Anne Bulkley, James Barry lived as a man
              throughout their medical education and career. In 1809, they
              travelled from London to Edinburgh by boat (~9 days of travel) to
              study medicine and graduated in 1812.
            </div>
          </div>
        </div>
      {:else if year === 1862}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/garrett_pic.jpg")}
              alt="Elizabeth Garrett"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              Elizabeth Garrett Anderson came to Edinburgh (most likely by
              train) in 1862, trying to enrol at the School of Medicine. She
              also tried to enrol at Universities of Cambridge, Glasgow, Oxford,
              and St Andrews, but was rejected by all.
            </div>
          </div>
        </div>
      {:else if year === 1867}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/first_english_classes_building.jpg")}
              alt="David Masson"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              The very first classes women could attend at the University (
              before the 'Edinburgh Seven') were David Masson's English
              Literature classes. A supporter of women's suffrage, Masson
              started teaching women in 1867 at Hopetoun Rooms (67-73 Queen
              Street).
            </div>
            <div class="milestone-map-section">
              <p class="milestone-map-credit">Map: {firstClassesMapCredit}</p>
            </div>
          </div>
        </div>
      {:else if year === 1869}
        <p class="edinburgh-forty-intro">
          The first women to matriculate at a university in the UK are well
          known as the 'Edinburgh Seven'. What's much less known is that
          actually 40 women matriculated at the School of Medicine between 1869
          and 1873.
        </p>
        <div class="edinburgh_forty">
          {#each orderedEdinburghFortyData as d (d.name)}
            <div
              class="edinburgh_forty-item"
              class:edinburgh_forty-item--edinburgh-seven={edinburghSevenOrder.has(
                d.name,
              )}
            >
              <div
                class="edinburgh_forty-circle"
                style:background-image={edinburghFortyImageUrl(d.img)}
              ></div>
              <div class="edinburgh_forty-name">{d.name}</div>
            </div>
          {/each}
        </div>
      {:else if year === 1870}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/surgeon_riot.jpg")}
              alt="Edinburgh Surgeon's Hall"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              A crowd of several hundred gathered as seven women medical
              students arrived for an anatomy exam, facing mud, abuse, and
              blocked gates.
            </div>
            <div class="milestone-map-section">
              <p class="milestone-map-credit">
                Map: {surgeonsHallRiotMapCredit}
              </p>
            </div>
          </div>
        </div>
      {:else if year === 1875}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/gayfield_house.jpg")}
              alt="John Gray McKendrick"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              After the 'Edinburgh Seven' were refused graduation, there were
              still professors who supported women in their pursuit to study
              medicine. One of them was John Gray McKendrick, who started
              teaching physiology to women at Gayfield House (18 East London
              Street) in 1875.
            </div>
            <div class="milestone-map-section">
              <p class="milestone-map-credit">
                Map: {physiologyStudentsMapCredit}
              </p>
            </div>
          </div>
        </div>
      {:else if year === 1886}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/esmw.jpg")}
              alt="Women Doctors"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              In 1886 Sophia Jex-Blake, the leading figure of the Edinburgh
              Seven/Forty, established the School of Medicine for Women where
              women could obtain a full medical education, being taught in the
              Extra-Mural School of Medicine because University teaching was
              closed to them.
            </div>
            <div class="milestone-map-section">
              <p class="milestone-map-credit">
                Map: {schoolForWomenMapCredit}
              </p>
            </div>
          </div>
        </div>
      {:else if year === 1889}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/ecmw.png")}
              alt="Women Doctors"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              Edinburgh College of Medicine for Women was established by The
              Scottish Association for the Medical Education of Women whose
              leading members included John Inglis, the father of Elsie Inglis.
              Elsie Inglis went on to become a leader in the suffrage movement
              and found the Scottish Women's Hospital.
            </div>
            <div class="milestone-map-section">
              <p class="milestone-map-credit">
                Map: {collegeForWomenMapCredit}
              </p>
            </div>
          </div>
        </div>
      {:else if year === 1911}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              id="women-doctors-1911-photo"
              class="milestone-image"
              src={publicUrl("img/photo_outside_efi.jpg")}
              alt="Women doctors outside the Edinburgh Futures Institute"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              By 1911, 300 women have qualified as doctors from the Edinburgh's
              School of Medicine and College of Medicine for Women. [Picture
              taken from outside the now Edinburgh Futures Institute, then the
              Edinburgh Royal Infirmary]
            </div>
          </div>
        </div>
      {:else if year === 1915}
        <div class="career-chart">
          {#if careerCountryCounts.length > 0}
            <div class="career-country-list">
              {#each careerCountryCounts as country (country.country)}
                <div class="career-country-row">
                  <div class="career-country-name">{country.country}</div>
                  <div
                    class="career-country-track"
                    aria-label={`${country.country}: ${country.count} women`}
                  >
                    <div
                      class="career-country-fill"
                      style:width={`${country.percent}%`}
                    ></div>
                  </div>
                  <div class="career-country-count">{country.count}</div>
                </div>
              {/each}
            </div>
          {:else}
            <div class="career-chart-empty">No 1915 career data</div>
          {/if}
        </div>
      {:else if year === 1916}
        <div class="milestone-card-text-only">
          <div class="milestone-card-title">
            After decades of separate medical education, wartime pressure and
            campaigning led the University to admit women to its medical
            classes, but even then, for several years, women were often taught
            separately from men.
          </div>
        </div>
        <!-- 1919 milestone temporarily disabled.
      {:else if year === womenDoctorsWarMilestoneYear}
        <div class="milestone-card-split-layout">
          <div class="milestone-card-split-half milestone-card-split-image">
            <img
              class="milestone-image"
              src={publicUrl("img/women_war.jpg")}
              alt="Women Doctors"
            />
          </div>
          <div class="milestone-card-split-half milestone-card-split-text">
            <div class="milestone-card-title">
              During the First World War, women doctors served in various
              capacities, including in military hospitals and overseas. The
              image above shows a group of women doctors during the war.
            </div>
          </div>
        </div>
      -->
        <!-- {:else if year === 1892}
        <div class="milestone-text">{milestoneLabels.get(year) ?? ""}</div>
        <img
          class="milestone-image"
          src={publicUrl("img/ordinance_1892.png")}
          alt="Women Admitted to Universities"
        /> -->
      {:else}
        <!-- {milestoneLabels.get(year) ?? ""} -->
      {/if}
    </div>
  {/each}
</main>

{#if !hasDismissedLoading}
  <AppLoadingScreen
    progress={loadingProgress}
    status={loadingStatus}
    ready={isReadyToEnter}
    error={loadError}
    onEnter={handleLoadingEnter}
    onRetry={handleRetry}
  />
{:else if introVisible}
  <AppIntro onExplore={handleExplore} onHome={handleIntroHome} />
{/if}

<style>
  main {
    width: 100%;
    height: 100vh;
    position: relative;
  }

  .site-logo {
    position: absolute;
    top: 2px;
    left: 2px;
    z-index: 3;
    height: 60px;
    pointer-events: none;
  }

  main:not(.experience-active) .site-logo,
  main:not(.experience-active) .page-title {
    opacity: 0;
  }

  .page-title {
    background-color: rgba(0, 0, 0, 0.5);
    position: absolute;
    top: 0px;
    left: 50%;
    z-index: 3;
    width: max-content;
    max-width: calc(100vw - 160px);
    margin: 0;
    color: #fff;
    font-size: clamp(16px, 2vw, 22px);
    font-weight: 400;
    line-height: 1.2;
    text-align: center;
    transform: translateX(-50%);
    pointer-events: none;
    border-radius: 0 0 5px 5px;
    padding: 5px;
  }

  .timeline-year-heading {
    position: absolute;
    left: 50%;
    bottom: 60px;
    z-index: 2;
    width: max-content;
    max-width: calc(100vw - 32px);
    margin: 0;
    padding: 5px;
    border-radius: 5px;
    background-color: rgba(0, 0, 0, 0.7);
    color: rgb(150, 150, 150);
    font-size: 12px;
    font-weight: 400;
    font-style: italic;
    line-height: 1.25;
    transform: translateX(-50%);
    pointer-events: none;
  }

  .map-image-credits {
    position: absolute;
    top: 0;
    left: 50%;
    z-index: 3;
    box-sizing: border-box;
    width: max-content;
    max-width: calc(100vw - 32px);
    height: 40px;
    padding: 0 16px;
    transform: translateX(-50%);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #ffffffdc;
    background-color: #000;
    font-size: 14px;
    line-height: 1.2;
    text-align: center;
    pointer-events: none;
    gap: 2px;
  }

  .milestone-card {
    position: absolute;
    z-index: 2;
    width: 30px;
    box-sizing: border-box;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: rgb(150, 150, 150);
    font-size: 10px;
    border-radius: 5px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
    background-color: #151c24;
    opacity: 0;
    pointer-events: none;
    overflow: hidden;
    border: 2px solid rgba(109, 109, 109, 0.24);
  }

  .milestone-card.is-active {
    opacity: 1;
  }

  .milestone-card.is-active {
    width: 400px;
    height: auto;
    max-height: calc(100vh - 65px - 65px);
    pointer-events: auto;
    flex-direction: column;
    align-items: stretch;
    justify-content: flex-start;
  }

  .milestone-card--edinburgh-forty.is-active {
    width: min(500px, calc(100vw - 32px));
    height: calc(100vh - 65px - 65px);
    container-type: size;
  }

  .milestone-card--women-doctors-1911.is-active {
    width: min(500px, calc(100vw - 32px));
  }

  .milestone-card-heading {
    flex: 0 0 auto;
    /* margin: 10px 10px 5px; */
    color: #fff;
    font-size: 18px;
    font-weight: 400;
    line-height: 1.1;
    text-align: center;
  }

  .edinburgh-forty-intro {
    flex: 0 0 auto;
    margin: 0 16px 8px;
    color: rgb(150, 150, 150);
    font-size: 14px;
    line-height: 1.25;
  }

  .milestone-card--split {
    padding: 0;
  }

  .milestone-card-text-only {
    width: 100%;
    flex: 1 1 auto;
    box-sizing: border-box;
    padding: 0 20px 20px;
    text-align: left;
  }

  .milestone-card--university-founded {
    align-items: stretch;
    justify-content: flex-start;
    padding: 0;
    background-color: #151c24;
  }

  .milestone-card--university-founded.is-active {
    width: min(90vw, 800px);
    height: calc(100vh - 65px - 65px);
    flex-direction: column;
    padding: 5px;
  }

  .milestone-card--university-founded .milestone-card-heading {
    margin: 5px 5px 8px;
  }

  .university-founded-card-image {
    width: calc(100% + 10px);
    min-height: 0;
    flex: 1 1 0;
    margin: 0 -5px;
    background-color: #000;
    background-position: center;
    background-repeat: no-repeat;
    background-size: contain;
  }

  .milestone-map-credit {
    flex: 0 0 auto;
    margin: 8px 5px 5px;
    color: rgb(150, 150, 150);
    font-size: 12px;
    font-style: italic;
    line-height: 1.25;
  }

  .milestone-map-section {
    margin-top: 16px;
    border-top: 1px solid rgba(150, 150, 150, 0.5);
  }

  .milestone-map-section .milestone-map-credit {
    margin: 8px 0 0;
  }

  .milestone-card--split .milestone-card-split-layout {
    width: 100%;
    height: auto;
    flex: 0 1 auto;
    min-height: 0;
    display: flex;
    flex-direction: column;
  }

  .milestone-card--split .milestone-card-split-half {
    min-height: 0;
    min-width: 0;
    box-sizing: border-box;
    display: flex;
    justify-content: center;
  }

  .milestone-card--split .milestone-card-split-image {
    flex: 0 0 auto;
    width: 100%;
    padding: 0.5rem;
    align-items: center;
  }

  .milestone-card--split .milestone-card-split-text {
    flex: 0 0 auto;
    flex-direction: column;
    align-items: stretch;
    padding: 20px;
    padding-top: 5px;
    text-align: left;
  }

  .milestone-card-title {
    font-size: 14px;
    text-align: left;
  }

  .milestone-image {
    width: 100%;
    height: auto;
    max-width: 100%;
    max-height: 45vh;
    object-fit: contain;
    display: block;
    flex: 0 0 auto;
    min-height: 0;
    border-radius: 3px;
  }

  .career-chart {
    width: 100%;
    height: auto;
    flex: 1 1 auto;
    min-height: 0;
    box-sizing: border-box;
    padding: 18px;
    overflow-y: auto;
    text-align: left;
  }

  .career-country-list {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  .career-country-row {
    display: grid;
    grid-template-columns: minmax(92px, 1fr) minmax(110px, 2.25fr) 28px;
    gap: 8px;
    align-items: center;
    min-height: 20px;
  }

  .career-country-name {
    min-width: 0;
    color: rgba(255, 255, 255, 0.9);
    font-size: 11px;
    line-height: 1.15;
    overflow-wrap: anywhere;
  }

  .career-country-track {
    height: 10px;
    overflow: hidden;
    background: rgba(255, 255, 255, 0.16);
  }

  .career-country-fill {
    height: 100%;
    min-width: 2px;
    background: #fff;
  }

  .career-country-count {
    color: rgba(255, 255, 255, 0.88);
    font-size: 11px;
    font-variant-numeric: tabular-nums;
    line-height: 1;
    text-align: right;
  }

  .career-chart-empty {
    color: rgba(255, 255, 255, 0.74);
    font-size: 12px;
    line-height: 1.4;
  }

  .edinburgh_forty {
    width: 100%;
    min-height: 0;
    box-sizing: border-box;
    flex: 1 1 0;
    display: grid;
    grid-template-columns: repeat(5, minmax(0, 1fr));
    grid-template-rows: repeat(8, minmax(0, 1fr));
    gap: clamp(1px, 0.4cqh, 3px);
    padding: clamp(1px, 0.4cqh, 3px);
    overflow: hidden;
  }

  .edinburgh_forty-item {
    min-width: 0;
    min-height: 0;
    box-sizing: border-box;
    display: grid;
    grid-template-rows: minmax(0, 1fr) auto;
    justify-items: center;
    align-items: center;
    gap: clamp(1px, 0.35cqh, 3px);
    padding: clamp(1px, 0.35cqh, 3px);
    background: #0d1116;
    border-radius: 3px;
    overflow: hidden;
  }

  .edinburgh_forty-item--edinburgh-seven {
    background: #495563;
  }

  .edinburgh_forty-circle {
    width: auto;
    height: 100%;
    max-width: 100%;
    min-height: 0;
    aspect-ratio: 1;
    border-radius: 50%;
    flex: 0 0 auto;
    background: rgba(56, 56, 56, 0.82);
    background-position: center;
    background-size: cover;
    background-repeat: no-repeat;
  }

  .edinburgh_forty-name {
    width: 100%;
    min-width: 0;
    color: #fff;
    font-size: clamp(5px, 1.5cqh, 10px);
    line-height: 1.05;
    text-align: center;
    overflow-wrap: anywhere;
    font-weight: 400;
  }

  .resume-button {
    position: absolute;
    top: 5px;
    right: 110px;
    z-index: 10;
    padding: 0.5rem 0.75rem;
    border: 1px solid rgba(255, 255, 255, 0.4);
    background: rgba(20, 20, 20, 0.75);
    color: #fff;
    font: inherit;
    cursor: pointer;
    border-radius: 3px;
  }
</style>
