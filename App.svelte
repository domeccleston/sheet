<script>
  import { onMount, onDestroy } from "svelte";
  import HyperFormula from "hyperformula";

  const options = { licenseKey: "gpl-v3" };

  const data = [
    ["item", "price", "quantity", "total"],
    ["apples", 0.69, 4, "=B2*C2"],
    ["bananas", 0.39, 6, "=B3*C3"],
    ["cantaloupes", 2.49, 2, "=D2*D3"],
    ["", "", "=SUM(C2:C4)", "=SUM(D2:D4)"]
  ];

  $: sheet = HyperFormula.buildFromArray(data, options);
  $: sheetValues = sheet.getSheetValues(0);

  function onKeyDown(event) {
    if (event.key === "Enter") {
      if (editable) {
        selected = { row: editable.row + 1, column: editable.column };
        commit(editable.row, editable.column);
        return;
      }
      if (selected.row !== null && selected.column !== null) {
        edit(selected.row, selected.column);
      }
    }
  }

  const expressions = [];

  onMount(() => {
    window.addEventListener("keydown", onKeyDown);
  });

  onDestroy(() => {
    window.removeEventListener("keydown", onKeyDown);
  });

  let selected = { row: null, column: null };

  let editable = { row: null, column: null };

  function numberToLetter(number) {
    return String.fromCharCode(97 + number);
  }

  function commit(row, column) {
    console.log("latest");
    editable = { row: null, column: null };
    window.getSelection().removeAllRanges();
  }

  function highlight(row, column) {
    if (selected.row === row && selected.column === column) return;
    if (editable.row === row && editable.column === column) return;
    commit();
    selected = { row, column };
  }

  function edit(row, column) {
    if (editable.row === row && editable.column === column) return;
    selected = { row: null, column: null };
    editable = { row, column };
  }

  function onChange(e, r, c) {
    console.log(e, r, c);
  }
</script>

<style>
  main {
    background: #4e505e;
    height: 100vh;

    font-family: Overpass;
    -webkit-font-smoothing: antialiased;
    text-align: center;
    padding-top: 40px;
  }

  table {
    background: #fff;
    margin-left: 40px;
    border-radius: 4px;
    box-shadow: 0px 5px 10px 0px rgba(0, 0, 0, 0.5);
  }

  table,
  th,
  td {
    border-collapse: collapse;
    box-sizing: border-box;
  }

  th,
  td {
    border: 1px solid gray;
    margin: 0;
    padding: 0;
    min-width: 70px;
    font-size: 14px;
  }

  .outline {
    border: 2px solid orange;
    cursor: text;
  }

  input {
    box-sizing: border-box;
    cursor: default;
    margin: 0;
    padding: 5px;
    height: 33px;
    width: 100%;
    border: 2px solid transparent;
    font-family: Overpass;
  }

  input:focus {
    outline: none;
  }

  .highlight {
    background: #ffdaca;
  }

  .row {
    position: relative;
  }

  .row-label {
    color: #fff;
    font-weight: 400;
    font-size: 12px;
    text-align: center;
    position: absolute;
    left: -20px;
    margin-top: 8px;
  }

  .heading {
    position: relative;
  }

  .text-formatting {
    text-align: left;
  }

  .number-formatting {
    text-align: right;
  }

  .column-label {
    position: absolute;
    bottom: 33px;
    color: #fff;
    font-size: 12px;
    font-weight: 400;
    text-transform: uppercase;
  }
</style>

<main>
  <table>
    {#each sheetValues as row, r}
      <tr class="row">
        <span class="row-label">{r + 1}</span>
        {#each sheetValues[r] as column, c}
          {#if r === 0}
            <th
              class="heading"
              class:number-formatting={typeof column === 'number'}
              class:text-formatting={typeof column !== 'number'}
              class:highlight={selected.row === r && selected.column === c}
              on:click={() => highlight(r, c)}
              on:dblclick={(e) => edit(r, c)}
            >
              <span class="column-label">
                {numberToLetter(c)}
              </span>
              <input
                class:outline={editable.row === r && editable.column === c}
                readonly={!editable || (editable.row !== r && editable.column !== c)}
                class:highlight={selected.row === r && selected.column === c}
                value={editable.row === r && editable.column === c ? data[r][c] : column}
                on:input={(event) => data[r][c] = event.target.value}
              >
            </th>
          {:else}
            <td
              class:number-formatting={typeof column === 'number'}
              class:text-formatting={typeof column !== 'number'}
              class:highlight={selected.row === r && selected.column === c}
              on:click={() => highlight(r, c)}
              on:dblclick={(e) => edit(r, c)}
            >
              <input
                class:outline={editable.row === r && editable.column === c}
                readonly={!editable || (editable.row !== r && editable.column !== c)}
                class:highlight={selected.row === r && selected.column === c}
                value={editable.row === r && editable.column === c ? data[r][c] : column}
                on:input={(event) => data[r][c] = event.target.value}
              />
            </td>
          {/if}
        {/each}
      </tr>
    {/each}
  </table>
</main>