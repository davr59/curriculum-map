<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import data from "./data.json";

  let result;
  let textAreaJson;

  const initialTextAreaJson = JSON.stringify(data);

  function onTextAreaChange() {
    updateMap();
  }

  function print() {
    window.print();
  }

  onMount(async () => {
    textAreaJson = initialTextAreaJson;
    updateMap();
  });

  const elementWidth = 6;
  const elementHeight = 1.75;
  const elementRadius = 1;
  const elementX = elementWidth * 0.4;
  const elementY = elementHeight / 4;
  const strokeWidth = 0.03;
  const fontSize = elementWidth / 15;
  const titleBX = elementWidth / 2;
  const titleBY = elementHeight / 3;
  const nameBX = elementWidth / 2;
  const nameBY = (elementHeight / 3) * 2;
  const maxLength = 22;

  function updateMap() {
    const data = JSON.parse(textAreaJson);
    const resultNode = getResultNode();
    const resultSvg = buildResultNodeSvg(resultNode);

    for (let index = 0; index < data.length; index++) {
      const group = resultSvg.append("g");
      group
        .selectAll()
        .data(data[index])
        .enter()
        .append("rect")
        .attr("width", elementWidth)
        .attr("height", elementHeight)
        .attr("x", (d, i) => index * (elementWidth + elementX))
        .attr("y", (d, i) => i * (elementHeight + elementY))
        .attr("rx", elementRadius)
        .attr("fill-opacity", 0)
        .attr("stroke", "black")
        .attr("stroke-width", strokeWidth)
        .transition()
        .attr("fill", "green")
        .duration(5000);
      group
        .selectAll()
        .data(data[index])
        .enter()
        .append("text")
        .attr("x", (d, i) => index * (elementWidth + elementX) + titleBX)
        .attr("y", (d, i) => i * (elementHeight + elementY) + titleBY)
        .attr("font-family", "monospace")
        .attr("font-size", (d, i) => fontSize)
        .attr("text-anchor", "middle")
        .attr("dominant-baseline", "middle")
        .text((d, i) => buildText(d.code));
      group
        .selectAll()
        .data(data[index])
        .enter()
        .append("text")
        .attr("x", (d, i) => index * (elementWidth + elementX) + nameBX)
        .attr("y", (d, i) => i * (elementHeight + elementY) + nameBY)
        .attr("font-family", "monospace")
        .attr("font-size", (d, i) => fontSize)
        .attr("text-anchor", "middle")
        .attr("dominant-baseline", "middle")
        .text((d, i) => buildText(d.name));
    }
  }

  function getResultNode() {
    const resultNode = d3.select(result);
    resultNode.html("");
    resultNode.style("width", "100vw").style("height", "100vh");
    return resultNode;
  }

  function buildResultNodeSvg(container) {
    return container.append("svg").attr("viewBox", "-0.1 -0.1 100 100");
  }

  function buildText(text) {
    return text.length <= maxLength
      ? text
      : `${text.substr(0, maxLength - 3)}...`;
  }
</script>

<style>
  main {
    text-align: center;
  }
  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
    margin: 8px 0 8px 0;
  }
  .divTextArea {
    margin: 0 16px 16px 16px;
  }
  textarea {
    width: 100%;
    margin: 0;
  }
  button {
    margin: 0 0 16px 0;
  }
  .divMap {
  }
  @media print {
    .noprint {
      display: none;
    }
  }
</style>

<main>
  <div class={'noprint'}>
    <form>
      <h1>MAPA DE CURSOS | COURSES MAP</h1>
      <div class={'divTextArea'}>
        <textarea
          placeholder="JSON"
          rows="8"
          bind:value={textAreaJson}
          on:keyup={onTextAreaChange} />
      </div>
      <button on:click={print}>Imprimir | Print</button>
    </form>
  </div>
  <div class={'divMap'}>
    <div bind:this={result} />
  </div>
</main>
