<template>
  <div id="viz" style="width: 100%; height: 100vh"></div>
</template>

<script setup>
import { onMounted } from "vue";
import Neovis from "neovis.js";

onMounted(() => {
  // Fake alerts data
  const alerts = [
    {
      alertId: "ALERT-001",
      hostId: "HOST-E095176FF0424EB4",
      title: "CPU usage high",
      description: "CPU usage over 90% for 10 minutes",
      severity: "high",
      isRootCause: true,
    },
    {
      alertId: "ALERT-002",
      hostId: "HOST-OTHERID123",
      title: "Disk failure",
      description: "Disk /dev/sda1 read errors",
      severity: "critical",
      isRootCause: false,
    },
    {
      alertId: "ALERT-003",
      hostId: "HOST-E095176FF0424EB4",
      title: "Memory leak detected",
      description: "Memory consumption increasing steadily",
      severity: "medium",
      isRootCause: false,
    },
  ];

  // Function to get root cause alerts by host ID
  const getRootCauseAlertsByHostId = (hostId) => {
    return alerts.filter((alert) => alert.hostId === hostId && alert.isRootCause === true);
  };

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
          size: 8,
          multi: "html",
          color: "#000",
          face: "arial",
          background: "transparent",
          strokeWidth: 0,
        },
        widthConstraint: {
          maximum: 50,
          minimum: 50,
        },
        scaling: {
          label: {
            enabled: true,
            min: 8,
            max: 12,
          },
        },
        color: {
          border: "#888",
          background: "#eee",
          highlight: {
            border: "#555",
            background: "#ddd",
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
      Node: {
        size: 10,
        [Neovis.NEOVIS_ADVANCED_CONFIG]: {
          function: {
            label: (node) => {
              // Hiển thị tên type nếu có, ví dụ: "KUBERNETES_CLUSTER"
              return node.properties?.type;
            },
            title: (node) => {
              return ` Entity ID: ${node.properties?.entityId || node.id}
                       Name: ${node.properties?.displayName || "-"}
                       Type: ${node.properties?.type || "-"}
                    `;
            },
          },
        },
      },
      // KubernetesCluster: {
      //   label: "Kubernetes",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "Kubernetes";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // EC2Instance: {
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "EC2";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // Host: {
      //   label: "HOST",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // Disk: {
      //   label: "DISK",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "DISK";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // ContainerGroup: {
      //   label: "Container Group",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "Container Group";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // ContainerGroupInstance: {
      //   label: "Container Group Instance",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "Container Group Instance";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // ProcessGroup: {
      //   label: "Process Group",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "Process Group";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // ProcessGroupInstance: {
      //   label: "Process Group Instance",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "Process Group Instance";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // Service: {
      //   label: "Service",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return node.properties.entityId;
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
      // AvailabilityZone: {
      //   label: "Availability Zone",
      //   size: 10,
      //   [Neovis.NEOVIS_ADVANCED_CONFIG]: {
      //     function: {
      //       label: (node) => {
      //         return "Availability Zone";
      //       },
      //       title: (node) => {
      //         return `ID: ${node.properties.entityId || node.id}`;
      //       },
      //     },
      //   },
      // },
    },
    initialCypher: `
      MATCH (n)-[r]->(m)
      RETURN n, r, m
    `,
  };

  const viz = new Neovis(config);

  viz.render();

  viz.registerOnEvent("completed", () => {
    const visJsNetwork = viz.network;
    if (!visJsNetwork) return;

    const allNodes = visJsNetwork.body.data.nodes.get();

    // Setup blinking animation for HOST nodes
    const flashingNodes = allNodes.filter((node) => {
      return node.label?.toUpperCase() === "HOST";
    });

    let start = null;
    const duration = 1000;

    function animateBlink(timestamp) {
      if (!start) start = timestamp;
      const elapsed = timestamp - start;

      const alpha = (Math.sin((elapsed / duration) * Math.PI * 2) + 1) / 2;

      const r = 255;
      const g = Math.round(255 * (1 - alpha));
      const b = Math.round(255 * (1 - alpha));

      const backgroundColor = `rgba(${r},${g},${b},1)`;

      flashingNodes.forEach((node) => {
        visJsNetwork.body.data.nodes.update({
          id: node.id,
          color: {
            background: backgroundColor,
            border: "#555",
          },
        });
      });

      requestAnimationFrame(animateBlink);
    }

    requestAnimationFrame(animateBlink);

    // Add click event listener for nodes
    // visJsNetwork.on("click", function (params) {
    //   if (params.nodes.length > 0) {
    //     const nodeId = params.nodes[0];
    //     const clickedNode = visJsNetwork.body.data.nodes.get(nodeId);

    //     // Check if clicked node is a HOST
    //     if (clickedNode && clickedNode.label?.toUpperCase() === "HOST") {
    //       // Get the host ID from node properties
    //       const hostId = clickedNode.title || clickedNode.id;

    //       // Find root cause alerts for this host
    //       const hostAlerts = getRootCauseAlertsByHostId(hostId);

    //       if (hostAlerts.length > 0) {
    //         console.log(`🚨 Root Cause Alerts for Host: ${hostId}`);
    //         console.log("=".repeat(50));

    //         hostAlerts.forEach((alert, index) => {
    //           console.log(`Root Cause Alert ${index + 1}:`);
    //           console.log(`  ID: ${alert.alertId}`);
    //           console.log(`  Title: ${alert.title}`);
    //           console.log(`  Description: ${alert.description}`);
    //           console.log(`  Severity: ${alert.severity.toUpperCase()}`);
    //           console.log("-".repeat(30));
    //         });

    //         // Also display summary for root cause alerts only
    //         const criticalCount = hostAlerts.filter((a) => a.severity === "critical").length;
    //         const highCount = hostAlerts.filter((a) => a.severity === "high").length;
    //         const mediumCount = hostAlerts.filter((a) => a.severity === "medium").length;

    //         console.log("📊 Root Cause Summary:");
    //         console.log(`  Total Root Cause Alerts: ${hostAlerts.length}`);
    //         console.log(`  Critical: ${criticalCount}, High: ${highCount}, Medium: ${mediumCount}`);
    //       } else {
    //         console.log(`✅ No root cause alerts found for Host: ${hostId}`);
    //       }
    //     }
    //   }
    // });

    // Hiển thị thông tin node khi click vào
    visJsNetwork.on("click", function (params) {
      if (params.nodes.length > 0) {
        const nodeId = params.nodes[0];
        const clickedNode = visJsNetwork.body.data.nodes.get(nodeId);

        console.log("📌 Node được click:");
        console.log(clickedNode); // ⚠️ In ra toàn bộ node để xem có thuộc tính gì

        // Nếu có node label là HOST, xử lý tiếp
        if (clickedNode && clickedNode.label?.toUpperCase() === "HOST") {
          const hostId = clickedNode.title || clickedNode.id;

          const hostAlerts = getRootCauseAlertsByHostId(hostId);

          if (hostAlerts.length > 0) {
            console.log(`🚨 Root Cause Alerts for Host: ${hostId}`);
            console.log("=".repeat(50));

            hostAlerts.forEach((alert, index) => {
              console.log(`Root Cause Alert ${index + 1}:`);
              console.log(`  ID: ${alert.alertId}`);
              console.log(`  Title: ${alert.title}`);
              console.log(`  Description: ${alert.description}`);
              console.log(`  Severity: ${alert.severity.toUpperCase()}`);
              console.log("-".repeat(30));
            });

            const criticalCount = hostAlerts.filter((a) => a.severity === "critical").length;
            const highCount = hostAlerts.filter((a) => a.severity === "high").length;
            const mediumCount = hostAlerts.filter((a) => a.severity === "medium").length;

            console.log("📊 Root Cause Summary:");
            console.log(`  Total Root Cause Alerts: ${hostAlerts.length}`);
            console.log(`  Critical: ${criticalCount}, High: ${highCount}, Medium: ${mediumCount}`);
          } else {
            console.log(`✅ No root cause alerts found for Host: ${hostId}`);
          }
        }
      }
    });
  });
});
</script>
