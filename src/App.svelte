<script lang="ts">
  import * as d3 from "d3";
  import * as pc from "d3-parliament-chart";
  import { onMount } from "svelte";

  import InputRange from "./InputRange.svelte";

  // doc: https://www.npmjs.com/package/d3-parliament-chart

  let parliamentGroups = [
    { seats: 193, color: "#A3BCF9" },
    { seats: 192, color: "pink" },
    { seats: 192, color: "blue" },
  ];

  let graphWidth = 1000;

  let groupsCollapsed = false;

  let seatsNumber: number;
  $: seatsNumber = Object.keys(parliamentGroups).reduce((previous, key) => {
    return previous + parliamentGroups[key].seats;
  }, 0);

  let seatsError = false;
  let displayGrid = false;
  let deltaSeats = 0;

  let graphLayout = {
    seatRadius: 7,
    rowHeight: 25,
    sectionsNumber: 53,
    sectionGap: 0,
  };

  onMount(() => {
    drawChart();
  });

  function updateChart() {
    drawChart();
    const circlesCount: number = d3
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
        .aggregatedData(parliamentGroups)
        .sections(graphLayout.sectionsNumber)
        .sectionGap(graphLayout.sectionGap)
        .seatRadius(graphLayout.seatRadius)
        .rowHeight(graphLayout.rowHeight)
        .debug(displayGrid)
    );
  }

  function addParliamentGroup() {
    parliamentGroups = [...parliamentGroups, { seats: 0, color: "pink" }];
  }

  function removeParliamentGroup(index: number) {
    parliamentGroups.splice(index, 1);
    parliamentGroups = parliamentGroups;
    drawChart();
  }

  function modifyGraphLayout(event: CustomEvent) {
    graphLayout[event.detail.name] = event.detail.value;
    updateChart();
  }

  function exportSvg() {
    if (document.querySelector("#chart svg .debug")) {
      document.querySelector("#chart svg .debug").remove();
    }
    displayGrid = false;
    // @ts-ignore
    svgExport.downloadSvg(
      document.querySelector("#chart svg"), // SVG DOM Element object to be exported. Alternatively, a string of the serialized SVG can be passed
      "parliament-chart", // chart title: file name of exported image
      { width: graphWidth, height: graphWidth / 2 } // options (optional, please see below for a list of option properties)
    );
  }
</script>

<main>
  <div id="chart">
    <svg width={graphWidth} height={graphWidth / 2} />
  </div>

  <div class="settings">
    <div>
      <div>
        <div class="settings-group">
          <h3 on:click={() => (groupsCollapsed = !groupsCollapsed)}>
            <span class="toggle {!groupsCollapsed ? 'open' : ''}">▶</span>
            Groupes
          </h3>
          {#if !groupsCollapsed}
            {#each parliamentGroups as group, index}
              <div>
                <h4>Groupe {index + 1}</h4>
                <div class="flex">
                  <label for="">Couleur</label>
                  <input
                    type="text"
                    bind:value={group.color}
                    on:input={updateChart}
                    min="0"
                    max="600"
                  />
                </div>
                <div class="flex">
                  <label for="">Nombre de sièges :</label>
                  <input
                    type="number"
                    bind:value={group.seats}
                    on:input={updateChart}
                    min="0"
                    max="600"
                  />
                </div>
                <p class="action" on:click={() => removeParliamentGroup(index)}>
                  Supprimer
                </p>
              </div>
            {/each}
            <p class="action" on:click={addParliamentGroup}>
              + Ajouter un groupe
            </p>
          {/if}
          <p>Nombre total de sièges : {seatsNumber}</p>
        </div>
        <div class="settings-group">
          <h3>Disposition</h3>
          <InputRange
            label="Rayon des cercles"
            name="seatRadius"
            min={5}
            max={20}
            bind:value={graphLayout.seatRadius}
            on:inputchange={modifyGraphLayout}
          />
          <InputRange
            label="Hauteur des rangs"
            name="rowHeight"
            min={0}
            max={100}
            bind:value={graphLayout.rowHeight}
            on:inputchange={modifyGraphLayout}
          />
          <InputRange
            label="Nombre de sections"
            name="sectionsNumber"
            min={1}
            max={100}
            bind:value={graphLayout.sectionsNumber}
            on:inputchange={modifyGraphLayout}
          />
          <InputRange
            label="Décalage entre les sections"
            name="sectionGap"
            min={0}
            max={40}
            bind:value={graphLayout.sectionGap}
            on:inputchange={modifyGraphLayout}
          />
        </div>
      </div>

      <div>
        {#if seatsError === true}
          <p class="alert">Oups, il manque {deltaSeats} sièges</p>
        {/if}
        <div>
          <label for="">Largeur du graphique (en px)</label>
          <input
            style="margin-top: 8px;"
            type="number"
            bind:value={graphWidth}
            on:input={updateChart}
            min="600"
            max="1600"
          />
        </div>
        <div class="flex" style="margin-top: 1em;">
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
    font-weight: 500;
  }

  h1,
  h2,
  h3,
  h4,
  h5,
  h6 {
    font-weight: 600;
  }

  :global(body) {
    min-height: 100vh;
  }

  :global(.debug line, .debug path) {
    stroke: #f0506e;
    opacity: 0.6;
  }

  #chart svg {
    margin: 4vh auto;
    transform: translateX(10%);
    display: block;
  }

  .settings {
    // max-width: 1000px;
    position: absolute;
    bottom: 4vh;
    left: 4vh;

    > div {
      display: grid;
      grid-template-columns: 1fr 1fr 1fr 1fr;
      align-items: flex-end;
      grid-gap: 80px;
    }

    .action {
      border-bottom: 1px solid;
      display: inline;
      cursor: pointer;
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

  button {
    margin-top: 1em;
    padding: 1em 2em;
    cursor: pointer;
  }

  .alert {
    color: #f0506e;
    background: #fef4f6;
    display: inline-block;
    padding: 0.5em 0.8em;
    font-weight: 500;
  }

  .settings-group + .settings-group {
    margin-top: 80px;
  }

  .toggle {
    display: inline-block;
    transition: transform 400ms;
  }

  .toggle.open {
    transform: rotate(90deg);
  }
</style>
