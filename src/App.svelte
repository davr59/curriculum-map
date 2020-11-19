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

  const FONT_FAMILY = "monospace";
  const FONT_SIZE_TITLE = 2.5;
  const FONT_SIZE_TEXT = 0.6;
  const CODE_DEFAULT_COLOR = "white";
  const DONE_DEFAULT_COLOR = "black";
  const STROKE_WIDTH = 0.05;
  const STROKE_WIDTH_DONE = 0.5;
  const MAX_TEXT_LENGTH = 22;
  const TRANSITION_DURATION = 3000;
  const MAX_COLUMNS_COUNT = 12;
  const MAX_ROWS_COUNT = 10;
  const COURSE_WIDTH = 8.5;
  const COURSE_HEIGHT = 5;
  const COURSE_RADIUS = 1;

  //falta
  const totalWidth = 99.5;
  const totalHeight = 71;

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

  //falta
  function refreshMap() {
    const data = _parseJson(textAreaJson);
    if (!data) {
      return;
    }
    const group = _buildGroup(svg);
    _drawTitle(group, data.title);
    let lastCoursesCoordinates = {};
    const columnsCount =
      data.courses.length > MAX_COLUMNS_COUNT
        ? MAX_COLUMNS_COUNT
        : data.courses.length;
    const x = totalWidth / data.courses.length;
    for (let columnIndex = 0; columnIndex < columnsCount; columnIndex++) {
      if (data.courses[columnIndex].length > MAX_ROWS_COUNT) {
        data.courses[columnIndex] = data.courses[columnIndex].slice(
          0,
          MAX_ROWS_COUNT
        );
      }
      const y = totalHeight / data.courses[columnIndex].length;
      _drawRect(
        group,
        data.courses[columnIndex],
        columnIndex,
        x,
        y,
        data.colors
      );
      _drawText(group, data.courses[columnIndex], columnIndex, x, y);
      if (isRequirementsEnabled) {
        const coursesCoordinates = _buildCoursesCoordinates(
          data.courses[columnIndex],
          columnIndex,
          x,
          y
        );
        _drawLine(
          group,
          data.courses[columnIndex],
          lastCoursesCoordinates,
          coursesCoordinates,
          columnIndex
        );
        lastCoursesCoordinates = coursesCoordinates;
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
      .attr("font-family", FONT_FAMILY)
      .attr("font-size", FONT_SIZE_TITLE);
  }

  function _drawRect(group, courses, columnIndex, x, y, colors) {
    return group
      .selectAll()
      .data(courses)
      .enter()
      .append("rect")
      .attr("width", COURSE_WIDTH)
      .attr("height", COURSE_HEIGHT)
      .attr("x", columnIndex * x + x / 8)
      .attr("y", (d, i) => i * y + y / 2)
      .attr("rx", COURSE_RADIUS)
      .attr("fill", d => _buildRectColor(d.code, colors))
      .attr("stroke", "black")
      .attr("stroke-width", STROKE_WIDTH)
      .transition()
      .attr("stroke", d =>
        isColoredEnabled && d.done && colors.done
          ? colors.done
          : DONE_DEFAULT_COLOR
      )
      .attr("stroke-width", d =>
        isCompletedEnabled && d.done ? STROKE_WIDTH_DONE : STROKE_WIDTH
      )
      .duration(TRANSITION_DURATION * (columnIndex + 1));
  }

  function _buildRectColor(code, colors) {
    if (!isColoredEnabled || !code || !colors) {
      return CODE_DEFAULT_COLOR;
    }
    const entry = Object.entries(colors).find(m => code.startsWith(m[0]));
    return (entry && entry[1]) || colors.codeDefault || CODE_DEFAULT_COLOR;
  }

  function _drawText(group, courses, columnIndex, x, y) {
    group
      .selectAll()
      .data(courses)
      .enter()
      .append("text")
      .attr("x", columnIndex * x + x / 8 + COURSE_WIDTH / 2)
      .attr("y", (d, i) => _buildTitleY(d.name, i, y))
      .attr("font-family", FONT_FAMILY)
      .attr("font-size", (d, i) => FONT_SIZE_TEXT)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .text((d, i) => d.code);
    group
      .selectAll()
      .data(courses)
      .enter()
      .append("text")
      .attr("x", columnIndex * x + x / 8 + COURSE_WIDTH / 2)
      .attr(
        "y",
        (d, i) => i * y + y / 2 + (COURSE_HEIGHT / 4) * 3 - FONT_SIZE_TEXT
      )
      .attr("font-family", FONT_FAMILY)
      .attr("font-size", (d, i) => FONT_SIZE_TEXT)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .text((d, i) => _buildText1(d.name));
    group
      .selectAll()
      .data(courses)
      .enter()
      .append("text")
      .attr("x", columnIndex * x + x / 8 + COURSE_WIDTH / 2)
      .attr("y", (d, i) => i * y + y / 2 + (COURSE_HEIGHT / 4) * 3)
      .attr("font-family", FONT_FAMILY)
      .attr("font-size", (d, i) => FONT_SIZE_TEXT)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .text((d, i) => _buildText2(d.name));
  }

  function _buildTitleY(name, i, y) {
    return name.length <= MAX_TEXT_LENGTH
      ? i * y + y / 2 + COURSE_HEIGHT / 2 - FONT_SIZE_TEXT
      : i * y + y / 2 + COURSE_HEIGHT / 4 + FONT_SIZE_TEXT / 2;
  }

  function _buildText1(text) {
    if (text.length <= MAX_TEXT_LENGTH) {
      return text;
    } else {
      let subtext = text.substr(0, MAX_TEXT_LENGTH);
      const blankIndex = subtext.lastIndexOf(" ");
      subtext = subtext.substr(0, blankIndex + 1).trim();
      return subtext;
    }
  }

  function _buildText2(text) {
    if (text.length <= MAX_TEXT_LENGTH) {
      return "";
    } else {
      let subtext = text.substr(0, MAX_TEXT_LENGTH);
      const blankIndex = subtext.lastIndexOf(" ");
      subtext = text.substr(blankIndex, text.length).trim();
      return subtext.length <= MAX_TEXT_LENGTH
        ? subtext
        : `${subtext.substr(0, MAX_TEXT_LENGTH - 3)}...`;
    }
  }

  function _buildCoursesCoordinates(courses, columnIndex, x, y) {
    const coursesCoordinates = {};
    for (let i = 0; i < courses.length; i++) {
      coursesCoordinates[courses[i].code.trim()] = {
        x1: columnIndex * x + x / 8,
        x2: columnIndex * x + x / 8 + COURSE_WIDTH,
        y: i * y + y / 2 + COURSE_HEIGHT / 2,
        done: courses[i].done
      };
    }
    return coursesCoordinates;
  }

  function _drawLine(
    group,
    courses,
    lastCoursesCoordinates,
    coursesCoordinates,
    columnIndex
  ) {
    if (
      !lastCoursesCoordinates ||
      !coursesCoordinates ||
      Object.keys(lastCoursesCoordinates) === 0 ||
      Object.keys(coursesCoordinates) === 0
    ) {
      return;
    }
    courses
      .filter(m => m.reqs)
      .forEach(m => {
        group
          .selectAll()
          .data(m.reqs.filter(n => lastCoursesCoordinates[n]))
          .enter()
          .append("line")
          .attr("x1", (d, i) => lastCoursesCoordinates[d.trim()].x2)
          .attr("y1", (d, i) => lastCoursesCoordinates[d.trim()].y)
          .attr("x2", (d, i) => coursesCoordinates[m.code.trim()].x1)
          .attr("y2", (d, i) => coursesCoordinates[m.code.trim()].y)
          .attr("stroke", "black")
          .attr("stroke-width", STROKE_WIDTH)
          .transition()
          .attr("x1", (d, i) =>
            isCompletedEnabled
              ? lastCoursesCoordinates[d.trim()].x2 +
                (lastCoursesCoordinates[d.trim()].done ? 0.25 : 0)
              : lastCoursesCoordinates[d.trim()].x2
          )
          .attr("x2", (d, i) =>
            isCompletedEnabled
              ? coursesCoordinates[m.code.trim()].x1 -
                (coursesCoordinates[m.code.trim()].done ? 0.25 : 0)
              : coursesCoordinates[m.code.trim()].x1
          )
          .duration(TRANSITION_DURATION * (columnIndex + 1));
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
            Error de formato JSON | JSON format error
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
