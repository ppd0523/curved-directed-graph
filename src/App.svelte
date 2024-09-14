<script lang="ts">
    import { onMount } from "svelte";
    import chroma from "chroma-js";
    import { v4 as uuid } from "uuid";

    import {MultiDirectedGraph} from "graphology";
    import forceAtlas2 from "graphology-layout-forceatlas2";
    import FA2Layout from "graphology-layout-forceatlas2/worker";
    import Sigma from "sigma";
    import EdgeCurveProgram, {EdgeCurvedArrowProgram} from "@sigma/edge-curve";
    import {EdgeArrowProgram} from "sigma/rendering";

    const graph = new MultiDirectedGraph();
    const sensibleSettings = forceAtlas2.inferSettings(graph);
    const layout = new FA2Layout(graph, {
        settings: sensibleSettings,
    });
    let containerElem = null;
    let renderer = null;

    onMount(()=>{

        // Create a sample graph
        graph.addNode("n1", { x: 0, y: 0, size: 5, color: chroma.random().hex() });
        graph.addNode("n2", { x: -5, y: 5, size: 5, color: chroma.random().hex() });
        graph.addNode("n3", { x: 5, y: 5, size: 5, color: chroma.random().hex() });
        graph.addNode("n4", { x: 0, y: 10, size: 5, color: chroma.random().hex() });
        graph.addNode("n5", { x: 0, y: 10, size: 5, color: chroma.random().hex() });
        graph.addEdge("n2", "n1", { size: 5, color: "yellow", curvature: 0.1 });
        graph.addEdge("n3", "n1", { size: 5, color: "blue", curvature: 0.3 });
        graph.addEdge("n4", "n1", { size: 5, color: "black", curvature: 0.4 });
        graph.addEdge("n5", "n1", { size: 5, color: "red", });
        graph.addEdge("n2", "n1", { size: 5, color: "gold", curvature: 0.2 });

        // Create the spring layout and start it
        layout.start();
        // Create the sigma
        // When clicking on the stage, we add a new node and connect it to the closest node

        renderer = new Sigma(graph, containerElem, {
            defaultEdgeType: "curved",
            edgeProgramClasses: {
                // straight: EdgeArrowProgram,
                curved: EdgeCurvedArrowProgram,
            },
        });
        renderer.setSetting("minArrowSize", 6);
        renderer.setSetting("maxArrowSize", 10);
        // renderer.on("clickStage", ({ event }: { event: { x: number; y: number } }) => {addNode(event.x, event.y)});
    })

    function addNode(x, y) {
        const coordForGraph = renderer.viewportToGraph({ x, y });

        // We create a new node
        const node = {
            ...coordForGraph,
            size: 5,
            color: chroma.random().hex(),
        };

        // We register the new node into graphology instance
        const id = uuid();
        graph.addNode(id, node);

        // We create the edges
        // closestNodes.forEach((e) => graph.addEdge(id, e.nodeId));
    }

</script>

<div id="sigma-container" bind:this={containerElem}></div>

<style>
    #sigma-container {
        width: 90vw;
        height: 90vh;
        border: 1px solid #ccc;
    }

</style>