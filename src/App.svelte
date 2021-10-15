<script>
  import * as d3 from "d3";
  import * as pc from "d3-parliament-chart";
  import { onMount } from "svelte";

  // doc: https://www.npmjs.com/package/d3-parliament-chart

  $: graphWidth = 1000;
  $: seatsNumber = 490;
  $: seatRadius = 12;
  $: rowHeight = 45;
  $: sectionsNumber = 49;
  $: sectionGap = 0;
  $: seatsError = false;
  $: displayGrid = true;
  $: deltaSeats = 0;

  onMount(() => {
    drawChart();
  });

  function updateChart() {
    drawChart();
    let circlesCount = d3
      .select(".parliament-chart")
      .selectAll("circle")
      .size();
    seatsError = circlesCount < seatsNumber;
    deltaSeats = seatsNumber - circlesCount;
  }

  function drawChart() {
    d3.select("svg").call(
      pc
        .parliamentChart()
        .width(graphWidth)
        .aggregatedData([{ seats: seatsNumber, color: "#A3BCF9" }])
        .sections(sectionsNumber)
        .sectionGap(sectionGap)
        .seatRadius(seatRadius)
        .rowHeight(rowHeight)
        .debug(displayGrid)
    );
  }

  function exportSvg() {
    document.querySelector("#chart svg .debug").remove();
    displayGrid = false;
    svgExport.downloadSvg(
      document.querySelector("#chart svg"), // SVG DOM Element object to be exported. Alternatively, a string of the serialized SVG can be passed
      "parliament-chart", // chart title: file name of exported image
      { width: graphWidth / 2, height: graphWidth / 4 } // options (optional, please see below for a list of option properties)
    );
  }
</script>

<main>
  <div id="chart">
    <svg width={graphWidth} height={graphWidth / 2} />
  </div>

  <div class="settings">
    {#if seatsError === true}
      <p class="alert">Oups, il manque {deltaSeats} sièges</p>
    {/if}
    <div>
      <div>
        <div class="flex">
          <label for="">Nombre de sièges :</label>
          <input
            class="uk-range"
            type="number"
            bind:value={seatsNumber}
            on:input={updateChart}
            min="0"
            max="600"
          />
        </div>
        <div>
          <label for="">Rayon des cercles : {seatRadius}</label>
          <input
            class="uk-range"
            type="range"
            bind:value={seatRadius}
            on:input={updateChart}
            min="5"
            max="20"
          />
        </div>
        <div>
          <label for="">Hauteur des rangs : {rowHeight}</label>
          <input
            class="uk-range"
            type="range"
            bind:value={rowHeight}
            on:input={updateChart}
            min="0"
            max="100"
          />
        </div>
        <div>
          <label for="">Nombre de sections : {sectionsNumber}</label>
          <input
            class="uk-range"
            type="range"
            bind:value={sectionsNumber}
            on:input={updateChart}
            min="1"
            max="180"
          />
        </div>
        <div>
          <label for="">Décalage entre les sections : {sectionGap}</label>
          <input
            class="uk-range"
            type="range"
            bind:value={sectionGap}
            on:input={updateChart}
            min="0"
            max="40"
          />
        </div>
      </div>
      <div>
        <div>
          <label for="">Largeur du graphique : {graphWidth}px</label>
          <input
            class="uk-range"
            type="range"
            bind:value={graphWidth}
            on:input={updateChart}
            min="600"
            max="1600"
          />
        </div>
        <div class="flex">
          <input
            id="grid"
            type="checkbox"
            bind:checked={displayGrid}
            on:change={updateChart}
          />
          <label for="grid">Afficher la grille</label>
        </div>
        <div>
          <button on:click={exportSvg}>Exporter</button>
        </div>
      </div>
    </div>
  </div>
</main>

<style lang="scss">
  * {
    box-sizing: border-box;
  }

  :global(body) {
    min-height: 100vh;
  }

  :global(.debug line, .debug path) {
    stroke: #f0506e;
  }

  #chart svg {
    margin: 4vh auto;
    display: block;
  }

  .settings {
    max-width: 1000px;
    position: absolute;
    bottom: 4vh;
    left: 4vh;

    > div {
      display: grid;
      grid-template-columns: 1fr 1fr;
      align-items: flex-end;
      grid-gap: 80px;
    }

    div + div {
      padding-top: 0.2em;
    }
  }

  .flex {
    display: flex;
    align-items: baseline;

    input[type="number"] {
      margin-left: 0.6em;
    }

    input[type="checkbox"] {
      margin-right: 0.6em;
    }
  }

  input[type="range"] {
    width: 200px;
    cursor: pointer;
  }

  button {
	  margin-top: 1em;
  }

  .alert {
    color: #f0506e;
    background: #fef4f6;
    display: inline-block;
    padding: 0.5em 0.8em;
    font-weight: 500;
  }
</style>
