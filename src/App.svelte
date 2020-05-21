<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let result;

  // https://brendansudol.com/writing/responsive-d3
  function responsivefy(svg) {
    let container = d3.select(svg.node().parentNode),
      width = parseInt(svg.style("width")),
      height = parseInt(svg.style("height")),
      aspect = width / height;
    svg
      .attr("viewBox", "0 0 " + width + " " + height)
      .attr("perserveAspectRatio", "xMinYMid")
      .call(resize);
    d3.select(window).on("resize." + container.attr("id"), resize);
    function resize() {
      let targetWidth = parseInt(container.style("width"));
      svg.attr("width", targetWidth);
      svg.attr("height", Math.round(targetWidth / aspect));
    }
  }

  function getResultNode(width, height) {
    const resultNode = d3.select(result);
    resultNode.attr("width", width).attr("height", height);
    return resultNode;
  }

  async function getData() {
    return d3.json("data.json");
  }

  function buildResultNodeSvg(container) {
    return container
      .append("svg")
      .attr("width", "100%")
      .attr("height", "100%")
      .call(responsivefy);
  }

  function buildCourses(
    group,
    data,
    width,
    height,
    blockIndex,
    blocksCount,
    blockCount
  ) {
    const xBlocks = 100 / blocksCount;
    const xBlockIndex = xBlocks * (blockIndex + 1);
    const x = xBlockIndex - xBlocks / 2 + "%";

    const y1 = 100 / blockCount;
    const y2 = (y1 * 3) / 4;
    const y3 = y1 / 2;
    group
      .selectAll()
      .data(data)
      .enter()
      .append("text")
      .transition()
      .duration(500)
      .text(c => c.code)
      .attr("x", x)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .attr("y", (d, i) => y1 * (i + 1) - y2 + "%");
    group
      .selectAll()
      .data(data)
      .enter()
      .append("text")
      .transition()
      .duration(500)
      .text(c => c.name)
      .attr("x", x)
      .attr("text-anchor", "middle")
      .attr("dominant-baseline", "middle")
      .attr("y", (d, i) => y1 * (i + 1) - y3 + "%");
  }

  onMount(async () => {
    const width = 800;
    const height = 600;

    const data = await getData();
    const resultNode = getResultNode(width, height);
    const resultSvg = buildResultNodeSvg(resultNode);

    for (let i = 0; i < data.length; i++) {
      const group = resultSvg.append("g");
      buildCourses(
        group,
        data[i],
        width,
        height,
        i,
        data.length,
        data[i].length
      );
      //group.selectAll().data(data).enter().append('rect').transition().duration(500).attr('width', 100).attr('height', (d, i) => 100).style('fill', 'white').attr('stroke', 'black');
    }

    //svg.append("foreignObject")
    // .attr("width", 480)
    // .attr("height", 500)
    //.append("xhtml:body")
    // .style("font", "14px 'Helvetica Neue'")
    // .html("<h1>An HTML Foreign Object in SVG</h1><p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec eu enim quam. Quisque nisi risus, sagittis quis tempor nec, aliquam eget neque. Nulla bibendum semper lorem non ullamcorper. Nulla non ligula lorem. Praesent porttitor, tellus nec suscipit aliquam, enim elit posuere lorem, at laoreet enim ligula sed tortor. Ut sodales, urna a aliquam semper, nibh diam gravida sapien, sit amet fermentum purus lacus eget massa. Donec ac arcu vel magna consequat pretium et vel ligula. Donec sit amet erat elit. Vivamus eu metus eget est hendrerit rutrum. Curabitur vitae orci et leo interdum egestas ut sit amet dui. In varius enim ut sem posuere in tristique metus ultrices.<p>Integer mollis massa at orci porta vestibulum. Pellentesque dignissim turpis ut tortor ultricies condimentum et quis nibh. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Integer euismod lorem vulputate dui pharetra luctus. Sed vulputate, nunc quis porttitor scelerisque, dui est varius ipsum, eu blandit mauris nibh pellentesque tortor. Vivamus ultricies ante eget ipsum pulvinar ac tempor turpis mollis. Morbi tortor orci, euismod vel sagittis ac, lobortis nec est. Quisque euismod venenatis felis at dapibus. Vestibulum dignissim nulla ut nisi tristique porttitor. Proin et nunc id arcu cursus dapibus non quis libero. Nunc ligula mi, bibendum non mattis nec, luctus id neque. Suspendisse ut eros lacus. Praesent eget lacus eget risus congue vestibulum. Morbi tincidunt pulvinar lacus sed faucibus. Phasellus sed vestibulum sapien.");

    //group.append('rect').attr('width', 150)
    //.attr('height', 100);
    //          .attr('x', 40)
    //            .attr('y', 100)
    //              .style('fill', 'white')
    //                .attr('stroke', 'black');
    //
  });
</script>

<style>

</style>

<main>
  <div bind:this={result} />
</main>
