<script>
  import world from "$data/110m.json";
  import data from "$data/iniciativas.json";
  import Legend from "$components/Legend.svelte";
  import Desplegable from "$components/Desplegable.svelte";
  import Globo from "$components/Globo.svelte";
  import { format } from "d3-format";
  import { scaleLinear } from "d3-scale";
  import { max } from "d3-array";

  let width = 400;
  $: height = width;

  // Mantengo tu lógica de resize intacta
  window.addEventListener("DOMContentLoaded", (event) => {
    function updateIframeHeight() {
      const el = document.documentElement;
      const rect = el.getBoundingClientRect();
      const styles = window.getComputedStyle(el);
      const margin =
        parseFloat(styles.marginTop) + parseFloat(styles.marginBottom);
      const height = Math.ceil(rect.height + margin);

      window.parent.postMessage(
        {
          type: "resize-iframe",
          value: height,
        },
        "*",
      );
    }
    updateIframeHeight();

    if (window.ResizeObserver) {
      new ResizeObserver(() => {
        updateIframeHeight();
      }).observe(document.documentElement);
    } else {
      window.addEventListener("load", updateIframeHeight);
      window.addEventListener("resize", updateIframeHeight);
    }

    window.addEventListener(
      "message",
      (event) => {
        if (event.data.type === "request-resize") {
          updateIframeHeight();
        }
      },
      false,
    );
  });

  const formatThousands = (n) => format(",")(n).replace(/,/g, ".");

  const uniqueCountries = Array.from(new Set(data.map((d) => d.pais))).sort(
    (a, b) => a.localeCompare(b),
  );

  $: desplegableOptions = uniqueCountries.map((name) => ({
    label: name,
    count: data
      .filter((d) => d.pais === name)
      .reduce((sum, d) => sum + d.iniciativas, 0),
  }));

  let selectedCountryName = "";
  let tooltipData = null;
  let iniciativas = null;
  let colorScale = scaleLinear()
    .domain([
      0,
      max(data.map((d) => d.iniciativas)) / 2,
      max(data.map((d) => d.iniciativas)),
    ])
    .range(["#ffffd9", "#ffda4d", "#ef4423"]);

  function handleTooltipChange(e) {
    tooltipData = e.detail.tooltipData;
    iniciativas = e.detail.iniciativas;
    colorScale = e.detail.colorScale;
  }
</script>

<div class="chart-container">
  <h1>
    Initiatives of South-South and Triangular Cooperation exchanged with each
    partner (2023-2024)
  </h1>
  <div class="legend-wrapper">
    <Legend {colorScale} {iniciativas} data={tooltipData} />
  </div>
  <div class="main-layout">
    <aside class="sidebar">
      <Desplegable
        options={desplegableOptions}
        bind:value={selectedCountryName}
        placeholder="Total Iberoamérica"
        formatCount={formatThousands}
      />
    </aside>

    <div class="viz-content">
      <Globo
        {world}
        {data}
        {selectedCountryName}
        on:tooltipChange={handleTooltipChange}
      />
    </div>
  </div>
</div>

<style>
  .chart-container {
    max-width: 1210px;
    margin: 0 auto;
    color: #3d3935;
  }

  h1 {
    text-align: center;
    color: #212c55;
    font-size: 20px;
    line-height: 26px;
    margin-bottom: 1.5rem;
  }

  /* Layout base (móvil): una columna */
  .main-layout {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .sidebar {
    width: 100%;
    z-index: 10;
  }

  .legend-wrapper {
    margin-bottom: 1rem;
    display: flex;
    justify-content: center;
  }

  /* Layout Desktop: menú al lado del globo */
  @media (min-width: 900px) {
    .main-layout {
      flex-direction: row;
    }

    .sidebar {
      width: 180px;
      flex-shrink: 0;
      position: sticky;
      top: 10px;
    }

    .viz-content {
      flex-grow: 1;
    }
  }
</style>
