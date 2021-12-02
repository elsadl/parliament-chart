<script lang="ts">
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let label: string;
  export let name: string;
  export let min: number;
  export let max: number;
  export let value: number;

  function updateValue() {
    dispatch("inputchange", { name: name, value: value });
  }
</script>

<div>
  <label for={name}>{label}: {value}</label>
  <input type="range" bind:value {min} {max} on:input={updateValue} />
</div>

<style>
  input[type="range"] {
    width: 200px;
    cursor: pointer;
  }
</style>
