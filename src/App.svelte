<script>
	import { onMount } from "svelte";
	import * as d3 from "d3";

  let resultNode;

	// https://brendansudol.com/writing/responsive-d3
	function responsivefy(svg) {
    var container = d3.select(svg.node().parentNode),
        width = parseInt(svg.style("width")),
        height = parseInt(svg.style("height")),
        aspect = width / height;
    svg.attr("viewBox", "0 0 " + width + " " + height)
        .attr("perserveAspectRatio", "xMinYMid")
        .call(resize);
    d3.select(window).on("resize." + container.attr("id"), resize);
    function resize() {
        var targetWidth = parseInt(container.style("width"));
        svg.attr("width", targetWidth);
        svg.attr("height", Math.round(targetWidth / aspect));
    }
	}

	function getResultNode() {
		return d3.select(resultNode);
	}

	async function getData() {
		return d3.json("data.json");
	}

	function buildResultNodeSvg(parentNode, width, height) {
		return parentNode.append('svg').attr('width', width).attr('height', height).call(responsivefy);
	}

	onMount(async () => {
		const width = 800;
		const height = 600;

		const data = await getData();
		const resultNode = getResultNode();
		const resultSvg = buildResultNodeSvg(resultNode, width, height);
		const group = resultSvg.append("g");
		group.selectAll().data(data).enter().append('text').transition().duration(500).text(c => c.name).attr('x', 50).attr('y', (d, i) => 50 + (i * 100));
		group.selectAll().data(data).enter().append('rect').transition().duration(500).attr('width', 50).attr('height', (d, i) => 50 + (i * 100));
							    //group.append('rect').attr('width', 150)
               //.attr('height', 100);
      //          .attr('x', 40)
    //            .attr('y', 100)
  //              .style('fill', 'white')
//                .attr('stroke', 'black');
//

	});
</script>

<main>
	<div bind:this={resultNode}></div>
</main>

<style>
</style>