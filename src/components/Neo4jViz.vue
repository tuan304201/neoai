<template>
  <div id="viz" style="width: 100%; height: 800px; border: 1px solid #ccc"></div>
</template>

<script setup>
import { onMounted } from "vue";
import Neovis from "neovis.js";

const expandedMap = new Map();

onMounted(() => {
  const config = {
    containerId: "viz",
    neo4j: {
      serverUrl: "bolt://192.168.1.109:7687",
      serverUser: "neo4j",
      serverPassword: "123123123",
    },
    visConfig: {
      nodes: {
        shape: "circle",
        font: {
          size: 12,
          multi: "html",
          color: "#000",
          face: "arial",
          background: "transparent",
          strokeWidth: 0,
        },
        widthConstraint: {
          maximum: 100,
        },
        scaling: {
          label: {
            enabled: true,
            min: 8,
            max: 10,
          },
        },
      },
      edges: {
        arrows: { to: { enabled: true } },
        width: 2,
        smooth: { enabled: true, type: "dynamic" },
      },
      physics: {
        enabled: true,
        forceAtlas2Based: {
          gravitationalConstant: -10,
          centralGravity: 0.001,
          springLength: 300,
          springConstant: 0.01,
        },
        minVelocity: 0.75,
        solver: "barnesHut",
        stabilization: {
          enabled: true,
          iterations: 1500,
          updateInterval: 10,
        },
      },
      interaction: {
        dragNodes: true,
        dragView: true,
        zoomView: true,
        hover: true,
      },
    },
    labels: {
      KubernetesCluster: { label: "id", size: 10 },
      EC2Instance: { label: "id", size: 10 },
      Host: { label: "id", size: 10 },
      Disk: { label: "id", size: 10 },
      ContainerGroup: { label: "id", size: 10 },
      ContainerGroupInstance: { label: "id", size: 10 },
      ProcessGroup: { label: "id", size: 10 },
      ProcessGroupInstance: { label: "id", size: 10 },
      Service: { label: "id", size: 10 },
      AvailabilityZone: { label: "id", size: 10 },
    },
    initialCypher: `
      MATCH (n:Host)
      RETURN n
    `,
  };

  const viz = new Neovis(config);

  viz.render();

  viz.registerOnEvent("clickNode", async (event) => {
    console.log("Click event:", event); // Debug log

    // Kiểm tra các cách có thể lấy nodeId từ event
    let nodeId = event.nodeId || event.node || event.nodes?.[0];

    // Nếu vẫn undefined, thử lấy từ event object khác
    if (!nodeId && event.pointer && event.pointer.DOM) {
      const network = viz.network;
      const nodeIds = network.getNodeAt(event.pointer.DOM);
      nodeId = nodeIds;
    }

    console.log("Node ID:", nodeId); // Debug log

    // Kiểm tra nodeId hợp lệ
    if (!nodeId || nodeId === undefined || nodeId === null) {
      console.error("NodeId is undefined or null");
      return;
    }

    const network = viz.network;
    const nodesDataset = network.body.data.nodes;
    const edgesDataset = network.body.data.edges;

    // Nếu node không còn trong graph (vì đã bị clear trước đó)
    if (!nodesDataset.get(nodeId)) {
      // Lấy nodeId cuối cùng đã được mở rộng
      const lastExpanded = [...expandedMap.entries()][0];
      if (!lastExpanded) return;

      const [oldNodeId, info] = lastExpanded;
      const label = info.label || "Host";

      // Thu gọn lại
      nodesDataset.clear();
      edgesDataset.clear();
      expandedMap.clear();

      await viz.updateWithCypher(`MATCH (n:\`${label}\`) RETURN n`);
      return;
    }

    // Nếu node đang mở → thu lại
    if (expandedMap.has(nodeId)) {
      const label = expandedMap.get(nodeId).label || "Host";

      nodesDataset.clear();
      edgesDataset.clear();
      expandedMap.clear();

      // Không cần nodeId khi thu lại, chỉ cần load lại tất cả nodes của label đó
      await viz.updateWithCypher(`MATCH (n:\`${label}\`) RETURN n`);
      return; // Thêm return để không chạy phần expand bên dưới
    } else {
      // Mở rộng
      const clickedNode = nodesDataset.get(nodeId);
      const label = clickedNode?.labels?.[0] || "Host";

      const beforeNodeIds = new Set(nodesDataset.getIds());
      const beforeEdgeIds = new Set(edgesDataset.getIds());

      // Đảm bảo nodeId là số hợp lệ trước khi dùng trong query
      const numericNodeId = parseInt(nodeId);
      if (isNaN(numericNodeId)) {
        console.error("NodeId is not a valid number:", nodeId);
        return;
      }

      console.log("Expanding node with ID:", numericNodeId); // Debug log

      await viz.updateWithCypher(`
        MATCH (n)-[r]-(m)
        WHERE id(n) = ${numericNodeId}
        RETURN n, r, m
      `);

      const afterNodeIds = new Set(nodesDataset.getIds());
      const afterEdgeIds = new Set(edgesDataset.getIds());

      const addedNodes = [...afterNodeIds].filter((id) => !beforeNodeIds.has(id));
      const addedEdges = [...afterEdgeIds].filter((id) => !beforeEdgeIds.has(id));

      // Lưu nodeId dạng string vào expandedMap để tránh lỗi type
      expandedMap.set(nodeId, { nodes: addedNodes, edges: addedEdges, label });
    }
  });
});
</script>
