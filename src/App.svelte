<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import data from "./data.json";

  let map;
  let svg;
  let textAreaJson;
  const initialTextAreaJson = JSON.stringify(data);

  const fontFamily = "monospace";
  const maxLabelLength = 22;
  const maxColumns = 12;
  const maxRows = 10;
  const transitionDuration = 3000;

  const labelFontSize = 0.6;
  const totalHeight = 71;
  const totalWidth = 99.5;
  const minWidth = 5;
  const minHeight = 10;
  const titleY = minHeight / 6 + 0.25;
  const nameY = minHeight / 3 - 0.25;
  const name2Y = minHeight / 3 + 0.5;
  const elementWidth = 8.5;
  const elementHeight = 5;
  const elementRadius = 1;
  const elementX = elementWidth * 0.3;
  const elementY = elementHeight / 4;
  const strokeWidth = 0.03;
  const titleBX = elementWidth / 2;
  const titleBY = elementHeight / 3;
  const nameBX = elementWidth / 2;
  const nameBY = (elementHeight / 3) * 2;

  function updateMap(isAnimated = true) {
    const data = JSON.parse(textAreaJson);
    drawTitle(svg, data.title);

    const columns =
      data.courses.length > maxColumns ? maxColumns : data.courses.length;
    for (let index = 0; index < columns; index++) {
      if (data.courses[index].length > maxRows) {
        data.courses[index] = data.courses[index].slice(0, maxRows);
      }

      const y = totalHeight / data.courses[index].length;
      const group = svg.append("g");

      drawRect(group, data.courses[index], index, y, data.colorsByCode);
      drawText(group, data.courses[index], index, y);
    }
  }

  function drawTitle(svg, title) {
    svg.html("");
    svg
      .append("text")
      .text(title || "")
      .attr("x", "50%")
      .attr("y", 1.5)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .attr("font-family", fontFamily)
      .attr("font-size", 2.5);
  }

  function drawRect(group, courses, index, y, colorsByCode) {
    return group
      .selectAll()
      .data(courses)
      .enter()
      .append("rect")
      .attr("width", elementWidth)
      .attr("height", elementHeight)
      .attr("x", index * (elementWidth + elementX))
      .attr("y", (d, i) => i * y + y / 2)
      .attr("rx", elementRadius)
      .attr("fill", d => buildRectColor(d.code, colorsByCode))
      .attr("stroke", "black")
      .attr("stroke-width", strokeWidth)
      .transition()
      .attr("fill", d => buildRectColor(d.code, colorsByCode))
      .duration(transitionDuration * (index + 1));
  }

  function buildRectColor(code, colorsByCode) {
    let defaultColor = "white";
    if (!code || !colorsByCode) {
      return defaultColor;
    }
    if (colorsByCode.default) {
      defaultColor = colorsByCode.default;
    }
    const entry = Object.entries(colorsByCode).find(m => code.startsWith(m[0]));
    return (entry && entry[1]) || defaultColor;
  }

  function drawText(group, courses, index, y) {
    group
      .selectAll()
      .data(courses)
      .enter()
      .append("text")
      .attr("x", index * (elementWidth + elementX) + titleBX)
      .attr("y", (d, i) => i * y + y / 2 + titleY)
      .attr("font-family", fontFamily)
      .attr("font-size", (d, i) => labelFontSize)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .text((d, i) => d.code);
    group
      .selectAll()
      .data(courses)
      .enter()
      .append("text")
      .attr("x", index * (elementWidth + elementX) + nameBX)
      .attr("y", (d, i) => i * y + y / 2 + nameY)
      .attr("font-family", fontFamily)
      .attr("font-size", (d, i) => labelFontSize)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .text((d, i) => buildText1(d.name, maxLabelLength));
    group
      .selectAll()
      .data(courses)
      .enter()
      .append("text")
      .attr("x", index * (elementWidth + elementX) + nameBX)
      .attr("y", (d, i) => i * y + y / 2 + name2Y)
      .attr("font-family", fontFamily)
      .attr("font-size", (d, i) => labelFontSize)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .text((d, i) => buildText2(d.name, maxLabelLength));
  }

  function buildText1(text, maxTextLength) {
    if (text.length <= maxTextLength) {
      return text;
    } else {
      let subtext = text.substr(0, maxTextLength);
      const blankIndex = subtext.lastIndexOf(" ");
      subtext = subtext.substr(0, blankIndex + 1).trim();
      return subtext;
    }
  }

  function buildText2(text, maxTextLength) {
    if (text.length <= maxTextLength) {
      return "";
    } else {
      let subtext = text.substr(0, maxTextLength);
      const blankIndex = subtext.lastIndexOf(" ");
      subtext = text.substr(blankIndex, text.length).trim();
      return subtext.length <= maxTextLength
        ? subtext
        : `${subtext.substr(0, maxTextLength - 3)}...`;
    }
  }

  function onTextAreaChange() {
    updateMap();
  }

  function animate() {}

  function print() {
    window.print();
  }

  function buildSvg() {
    const mapNode = d3.select(map);
    mapNode.style("width", "100vw").style("height", "100vh");
    return mapNode.append("svg").attr("viewBox", "0 0 100 75");
  }

  onMount(async () => {
    textAreaJson = initialTextAreaJson;
    svg = buildSvg();
    updateMap();
  });
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
    margin: 8px 0 16px 0;
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
  .buttonAnimate {
    margin-right: 1px;
  }
  .divMap {
  }
  @media print {
    .noprint {
      display: none;
    }
    @page {
      size: landscape;
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
      <button class="buttonAnimate" on:click={animate}>Animar | Animate</button>
      <button on:click={print}>Imprimir | Print</button>
    </form>
  </div>
  <div class={'divMap'}>
    <div bind:this={map} />
  </div>
</main>
