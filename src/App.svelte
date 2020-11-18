<script>
  import { afterUpdate, onMount } from "svelte";
  import * as d3 from "d3";
  import data from "./data.json";

  let map;
  let svg;
  let textAreaJson;
  let isColoredEnabled;
  let isCompletedEnabled;
  let isRequirementsEnabled;
  let isJsonError;

  const fontFamily = "monospace";
  const maxColumnsCount = 12;
  const maxRowsCount = 10;
  const maxTextLength = 22;
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
  const doneStrokeWidth = 0.03 * 10;
  const titleBX = elementWidth / 2;
  const titleBY = elementHeight / 3;
  const nameBX = elementWidth / 2;
  const nameBY = (elementHeight / 3) * 2;

  onMount(async () => {
    textAreaJson = JSON.stringify(data, undefined, "\t");
    isColoredEnabled = true;
    isCompletedEnabled = true;
    isRequirementsEnabled = true;
    isJsonError = false;
    svg = _buildSvg();
  });

  afterUpdate(() => {
    refreshMap();
  });

  function print() {
    window.print();
  }

  function refreshMap() {
    const data = _parseJson(textAreaJson);
    if (!data) {
      return;
    }
    const group = _buildGroup(svg);
    _drawTitle(group, data.title);
    let oldLines = {};
    const columnsCount =
      data.courses.length > maxColumnsCount
        ? maxColumnsCount
        : data.courses.length;
    for (let index = 0; index < columnsCount; index++) {
      if (data.courses[index].length > maxRowsCount) {
        data.courses[index] = data.courses[index].slice(0, maxRowsCount);
      }
      const y = totalHeight / data.courses[index].length;
      _drawRect(group, data.courses[index], index, y, data.colorsByCode);
      _drawText(group, data.courses[index], index, y);
      if (isRequirementsEnabled) {
        const newLines = _buildLines(data.courses[index], index, y);
        _drawLine(group, data.courses[index], oldLines, newLines);
        oldLines = newLines;
      }
    }
  }

  function _buildSvg() {
    const mapNode = d3.select(map);
    mapNode.style("width", "100vw").style("height", "100vh");
    return mapNode.append("svg").attr("viewBox", "0 0 100 75");
  }

  function _parseJson(json) {
    try {
      const data = JSON.parse(json);
      isJsonError = false;
      return data;
    } catch (e) {
      console.log(e);
      isJsonError = true;
      return undefined;
    }
  }

  function _buildGroup(svg) {
    svg.html("");
    return svg.append("g");
  }

  function _drawTitle(group, title) {
    group
      .append("text")
      .text(title || "")
      .attr("x", "50%")
      .attr("y", 1.5)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .attr("font-family", fontFamily)
      .attr("font-size", 2.5);
  }

  function _drawRect(group, courses, index, y, colorsByCode) {
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
      .attr("fill", d => _buildRectColor(d.code, colorsByCode))
      .attr("stroke", "black")
      .attr("stroke-width", strokeWidth)
      .transition()
      .attr("stroke", d => (d.done ? "green" : "black"))
      .attr("stroke-width", d => (d.done ? doneStrokeWidth : strokeWidth))
      .duration(transitionDuration * (index + 1));
  }

  function _buildRectColor(code, colorsByCode) {
    let defaultColor = "white";
    if (!isColoredEnabled || !code || !colorsByCode) {
      return defaultColor;
    }
    if (colorsByCode.default) {
      defaultColor = colorsByCode.default;
    }
    const entry = Object.entries(colorsByCode).find(m => code.startsWith(m[0]));
    return (entry && entry[1]) || defaultColor;
  }

  function _drawText(group, courses, index, y) {
    group
      .selectAll()
      .data(courses)
      .enter()
      .append("text")
      .attr("x", index * (elementWidth + elementX) + titleBX)
      .attr("y", (d, i) =>
        d.name.length <= maxTextLength
          ? i * y + y / 2 + titleY
          : i * y + y / 2 + titleY - 0.5
      )
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
      .text((d, i) => _buildText1(d.name, maxTextLength));
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
      .text((d, i) => _buildText2(d.name, maxTextLength));
  }

  function _buildText1(text, maxTextLength) {
    if (text.length <= maxTextLength) {
      return text;
    } else {
      let subtext = text.substr(0, maxTextLength);
      const blankIndex = subtext.lastIndexOf(" ");
      subtext = subtext.substr(0, blankIndex + 1).trim();
      return subtext;
    }
  }

  function _buildText2(text, maxTextLength) {
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

  function _buildLines(courses, index, y) {
    const lines = {};
    for (let i = 0; i < courses.length; i++) {
      lines[courses[i].code.trim()] = {
        x1: index * (elementWidth + elementX),
        x2: index * (elementWidth + elementX) + elementWidth,
        y: i * y + y / 2 + elementHeight / 2
      };
    }
    return lines;
  }

  function _drawLine(group, courses, oldLines, newLines) {
    if (
      !oldLines ||
      !newLines ||
      Object.keys(oldLines) === 0 ||
      Object.keys(newLines) === 0
    ) {
      return;
    }
    courses
      .filter(m => m.reqs)
      .forEach(m => {
        group
          .selectAll()
          .data(m.reqs.filter(n => oldLines[n]))
          .enter()
          .append("line")
          .attr("x1", (d, i) => oldLines[d.trim()].x2)
          .attr("y1", (d, i) => oldLines[d.trim()].y)
          .attr("x2", (d, i) => newLines[m.code.trim()].x1)
          .attr("y2", (d, i) => newLines[m.code.trim()].y)
          .attr("stroke", "black")
          .attr("stroke-width", strokeWidth);
      });
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
    margin: 8px 0 16px 0;
  }
  .divTextArea {
    margin: 0 16px 16px 16px;
  }
  textarea {
    width: 100%;
    margin: 0;
  }
  .jsonErrorTextArea {
    box-shadow: 0 0 0 2px #ff3e00;
    outline: none;
  }
  .jsonErrorMessage {
    color: #ff3e00;
  }
  .divCheckBox {
    display: flex;
    justify-content: center;
    margin: 0 0 16px 0;
  }
  .separatorLabel {
    margin-right: 10px;
  }
  button {
    margin: 0 0 16px 0;
  }
  .separatorButton {
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
          class:jsonErrorTextArea={isJsonError}
          placeholder="JSON"
          rows="8"
          bind:value={textAreaJson} />
        {#if isJsonError}
          <span class="jsonErrorMessage">
            Error del formato JSON | JSON format error
          </span>
        {:else}Datos en formato JSON | Data in JSON format{/if}
      </div>
      <div class="divCheckBox">
        <label class="separatorLabel">
          <input type="checkbox" bind:checked={isColoredEnabled} />
          Mostrar colores | Show colors
        </label>
        <label class="separatorLabel">
          <input type="checkbox" bind:checked={isCompletedEnabled} />
          Mostrar completado | Show completed
        </label>
        <label>
          <input type="checkbox" bind:checked={isRequirementsEnabled} />
          Mostrar requisitos | Show requirements
        </label>
      </div>
      {#if isCompletedEnabled}
        <button type="button" class="separatorButton" on:click={refreshMap}>
          Animar | Animate
        </button>
      {/if}
      <button type="button" on:click={print}>Imprimir | Print</button>
    </form>
  </div>
  <div class={'divMap'}>
    <div bind:this={map} />
  </div>
</main>
