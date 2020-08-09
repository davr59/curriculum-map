<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let result;

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

  async function getData() {
    return d3.json("data.json");
  }

  function getResultNode() {
    const resultNode = d3.select(result);
    resultNode.style("width", "100vw").style("height", "100vh");
    return resultNode;
  }

  function buildResultNodeSvg(container) {
    return container.append("svg").attr("viewBox", "-0.1 -0.1 100 100");
  }

  function buildFontSize(size) {
    return fontSize;
  }

  onMount(async () => {
    const data = await getData();
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
        .attr("stroke-width", strokeWidth);
      group
        .selectAll()
        .data(data[index])
        .enter()
        .append("text")
        .attr("x", (d, i) => index * (elementWidth + elementX) + titleBX)
        .attr("y", (d, i) => i * (elementHeight + elementY) + titleBY)
        .attr("font-family", "monospace")
        .attr("font-size", (d, i) => buildFontSize(d.code.length))
        .attr("text-anchor", "middle")
        .attr("dominant-baseline", "middle")
        .text((d, i) => d.code);
      group
        .selectAll()
        .data(data[index])
        .enter()
        .append("text")
        .attr("x", (d, i) => index * (elementWidth + elementX) + nameBX)
        .attr("y", (d, i) => i * (elementHeight + elementY) + nameBY)
        .attr("font-family", "monospace")
        .attr("font-size", (d, i) => buildFontSize(d.name.length))
        .attr("text-anchor", "middle")
        .attr("dominant-baseline", "middle")
        .text((d, i) => d.name);
    }
  });
</script>

<style>

</style>

<main>
  <div bind:this={result} />
</main>
