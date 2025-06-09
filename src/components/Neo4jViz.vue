<template>
  <div class="flex flex-col h-screen bg-white text-gray-800 font-sans">
    <!-- Header -->
    <div class="flex-shrink-0 bg-gray-50 border-b border-gray-200 px-4 py-2">
      <span class="text-lg font-semibold text-gray-700">Graph</span>
    </div>

    <!-- Main Content Area -->
    <div class="flex flex-grow overflow-hidden">
      <!-- Graph Visualization Area -->
      <div class="flex-grow relative">
        <div id="viz" class="w-full h-full"></div>
        <div
          v-if="isLoading"
          class="absolute top-0 left-0 w-full h-full flex items-center justify-center bg-white/70 z-10"
        >
          <div class="bg-white p-5 rounded-lg shadow-md text-lg text-center">
            <svg
              class="animate-spin h-8 w-8 text-gray-700 mx-auto mb-3"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
            >
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
              ></path>
            </svg>
            {{ statusMessage }}
          </div>
        </div>
      </div>

      <!-- Results Overview Panel -->
      <div class="flex-shrink-0 w-80 bg-gray-50 border-l border-gray-200 p-4 overflow-y-auto">
        <!-- Chế độ xem chi tiết node -->
        <div v-if="selectedNodeDetails">
          <h2 class="text-lg font-bold mb-3 text-gray-800">Node details</h2>
          <div class="flex flex-wrap gap-2 mb-4">
            <span
              v-for="label in selectedNodeDetails.labels"
              :key="label"
              class="px-3 py-1 text-sm font-medium rounded-full cursor-default text-white"
              :style="{ backgroundColor: getAssignedColor(label) }"
            >
              {{ label }}
            </span>
          </div>
          <table class="w-full text-sm text-left mb-4">
            <thead class="bg-gray-200">
              <tr>
                <th class="p-2 font-semibold">Key</th>
                <th class="p-2 font-semibold">Value</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(value, key) in selectedNodeDetails.properties" :key="key" class="border-b border-gray-200">
                <td class="p-2 font-mono align-top">{{ key }}</td>
                <td class="p-2 font-mono break-words">{{ value }}</td>
              </tr>
            </tbody>
          </table>

          <div v-if="selectedNodeDetails.alerts && selectedNodeDetails.alerts.length > 0" class="mt-4">
            <h3 class="text-md font-bold mb-2 text-gray-800">Associated Alerts</h3>
            <div
              v-for="(alert, index) in selectedNodeDetails.alerts"
              :key="index"
              class="text-sm border-t border-gray-200 p-2 bg-red-50 rounded"
            >
              <p class="font-semibold text-red-800 break-words">{{ alert.title }}</p>
              <div class="font-mono text-xs text-gray-600 mt-2 space-y-1 break-words">
                <p><strong>Entity ID:</strong> {{ alert.entityId }}</p>
                <p><strong>Path:</strong> {{ alert.path }}</p>
              </div>
            </div>
          </div>
        </div>

        <!-- Chế độ xem tổng quan (mặc định) -->
        <div v-else>
          <h2 class="text-lg font-bold mb-3 text-gray-800">Results overview</h2>
          <div class="mb-6">
            <!-- ** MỚI: Ô tìm kiếm Node ** -->
            <div class="mb-3">
              <input
                type="text"
                v-model="nodeFilterText"
                placeholder="Filter nodes..."
                class="w-full px-3 py-2 text-sm border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
            <h3 class="font-semibold text-gray-600 mb-2">
              Nodes ({{ filteredNodeLabelCounts.length }}/{{ totalNodes }})
            </h3>
            <div class="flex flex-wrap gap-2">
              <span
                @click="clearGraph()"
                class="px-3 py-1 text-sm font-medium rounded-full cursor-pointer transition-all bg-gray-200 text-gray-700 hover:bg-gray-300"
              >
                Clear Graph
              </span>
              <span
                @click="showAllNodes()"
                class="px-3 py-1 text-sm font-medium rounded-full cursor-pointer transition-all bg-gray-200 text-gray-700 hover:bg-gray-300"
              >
                Show All
              </span>
              <!-- ** SỬA ĐỔI: Sử dụng danh sách đã lọc ** -->
              <span
                v-for="item in filteredNodeLabelCounts"
                :key="item.label"
                @click="toggleNodesByType(item.label)"
                class="px-3 py-1 text-sm font-medium rounded-full cursor-pointer text-white transition-all"
                :style="{ backgroundColor: item.color }"
                :class="{
                  'opacity-40 hover:opacity-70': activeNodeLabels.size > 0 && !activeNodeLabels.has(item.label),
                  'ring-2 ring-offset-2 ring-blue-500': activeNodeLabels.has(item.label),
                }"
              >
                {{ item.label }} ({{ item.count }})
              </span>
            </div>
          </div>
          <div>
            <h3 class="font-semibold text-gray-600 mb-2">Relationships ({{ totalRelationships }})</h3>
            <div class="flex flex-wrap gap-2">
              <span
                v-for="item in relationshipTypeCounts"
                :key="item.type"
                class="px-3 py-1 text-sm font-medium rounded-full cursor-default text-white"
                :style="{ backgroundColor: item.color }"
              >
                {{ item.type }} ({{ item.count }})
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onUnmounted, nextTick, computed } from "vue";
import Neovis from "neovis.js";
import neo4j from "neo4j-driver";

// --- State ---
const statusMessage = ref("Đang khởi tạo...");
const isLoading = ref(true);
const nodeLabelCounts = ref([]);
const relationshipTypeCounts = ref([]);
const activeNodeLabels = ref(new Set());
const selectedNodeDetails = ref(null);
let animationFrameId = null;
const nodeFilterText = ref("");

// --- Dữ liệu cảnh báo (Alert) ---
const incidents = ref([
  {
    incident_name: "Service Degradation: Success Rate Triggered",
    incident_time: "2025-06-03T07:07:14.572000+00:00",
    alerts: [
      {
        source: "Grafana",
        title:
          "[UAT] SmartCredit JVM Heap Usage Over 70% {alertname=[UAT] SmartCredit JVM Heap Usage Over 70%, grafana_folder=Smart Credit, instance=	tcb-pt-rdb-heimdall-proxy-baseline, job=cf-deploy} - B=70.046663, C=1.000000",
        alert_id: 133,
        path: "s3://vncs-aiops-data/telemetry/grafana/ingest_year=2025/ingest_month=06/ingest_day=03/ingest_hour=07/run-1748934502023-part-r-00000",
        entityId: "HOST-E095176FF0424EB4",
        displayName: "tcb-pt-rdb-heimdall-proxy-baseline",
      },

      {
        source: "Solarwinds",
        title: "ALERT: Node DX-PA5280-01 is Down",
        alert_id: "ALERT: Node DX-PA5280-01 is Down",
        path: "s3://vncs-aiops-data/telemetry/solarwinds/ingest_year=2025/ingest_month=06/ingest_day=06/ingest_hour=03/run-1749180802733-part-r-00000",
        entityId: "3456-dfgbhn-bvc-sdfg",
        displayName: "DX-PA5280-01",
      },
    ],
  },
]);

// --- Computed Properties ---
const totalNodes = computed(() => nodeLabelCounts.value.reduce((sum, item) => sum + item.count, 0));
const totalRelationships = computed(() => relationshipTypeCounts.value.reduce((sum, item) => sum + item.count, 0));

const alertsByDisplayName = computed(() => {
  const map = new Map();
  incidents.value.forEach((incident) => {
    incident.alerts.forEach((alert) => {
      if (alert.displayName) {
        if (!map.has(alert.displayName)) map.set(alert.displayName, []);
        map.get(alert.displayName).push(alert);
      }
    });
  });
  return map;
});
const alertedEntityNames = computed(() => new Set(alertsByDisplayName.value.keys()));

// ** MỚI: Computed property để lọc danh sách node **
const filteredNodeLabelCounts = computed(() => {
  if (!nodeFilterText.value) {
    return nodeLabelCounts.value;
  }
  const filter = nodeFilterText.value.toLowerCase();
  return nodeLabelCounts.value.filter((item) => item.label.toLowerCase().includes(filter));
});

let neovisInstance = null;

// --- Neo4j Connection ---
const NEO4J_URL = "neo4j://192.168.1.109:7687";
const NEO4J_USER = "neo4j";
const NEO4J_PASSWORD = "123123123";

// --- Helper Functions ---
const colorPalette = [
  "#588BAE",
  "#FF6961",
  "#6D3B47",
  "#6EB4D1",
  "#CF4647",
  "#4D545E",
  "#E4A66B",
  "#62929E",
  "#934553",
  "#94C5CC",
  "#F4C2C2",
  "#7E8A97",
];
let colorIndex = 0;
const assignedColors = {};

function getAssignedColor(item) {
  if (!assignedColors[item]) {
    assignedColors[item] = colorPalette[colorIndex % colorPalette.length];
    colorIndex++;
  }
  return assignedColors[item];
}

const startBlinkingAnimation = (nodeIds) => {
  stopBlinkingAnimation();
  if (!neovisInstance?.network || nodeIds.length === 0) return;
  const networkNodes = neovisInstance.network.body.data.nodes;
  const originalColors = new Map();
  nodeIds.forEach((id) => {
    const node = networkNodes.get(id);
    if (node) originalColors.set(id, node.color?.background || "#CCCCCC");
  });
  const duration = 1000;
  let start = null;
  function animate(timestamp) {
    if (!start) start = timestamp;
    const elapsed = timestamp - start;
    const alpha = 0.5 * (1 + Math.sin((2 * Math.PI * elapsed) / duration));
    const updates = nodeIds.map((id) => {
      const originalColor = originalColors.get(id);
      const blinkColor = "#FF4136";
      const newColor = alpha > 0.5 ? blinkColor : originalColor;
      return { id, color: { background: newColor, border: blinkColor } };
    });
    if (updates.length > 0) networkNodes.update(updates);
    animationFrameId = requestAnimationFrame(animate);
  }
  animationFrameId = requestAnimationFrame(animate);
};

const stopBlinkingAnimation = () => {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId);
    animationFrameId = null;
  }
};

// --- Core Logic ---
const fetchOverviewData = async () => {
  isLoading.value = true;
  statusMessage.value = "Đang tải dữ liệu tổng quan...";
  const driver = neo4j.driver(NEO4J_URL, neo4j.auth.basic(NEO4J_USER, NEO4J_PASSWORD));
  const labelsQuery = `MATCH (n) UNWIND labels(n) AS label RETURN label, count(n) AS count ORDER BY count DESC`;
  const relsQuery = `MATCH ()-[r]->() RETURN type(r) AS type, count(r) AS count ORDER BY count DESC`;
  try {
    const [labelsResult, relsResult] = await Promise.all([
      driver.session().run(labelsQuery),
      driver.session().run(relsQuery),
    ]);
    nodeLabelCounts.value = labelsResult.records.map((r) => ({
      label: r.get("label"),
      count: r.get("count").toNumber(),
      color: getAssignedColor(r.get("label")),
    }));
    relationshipTypeCounts.value = relsResult.records.map((r) => ({
      type: r.get("type"),
      count: r.get("count").toNumber(),
      color: getAssignedColor(r.get("type")),
    }));
  } catch (e) {
    console.error("Lỗi khi tải dữ liệu tổng quan:", e);
    statusMessage.value = "Lỗi tải dữ liệu tổng quan.";
  } finally {
    isLoading.value = false;
    statusMessage.value = "Sẵn sàng.";
    await driver.close();
  }
};

const renderActiveGraph = () => {
  stopBlinkingAnimation();
  if (!neovisInstance) return;
  const labels = Array.from(activeNodeLabels.value);
  if (labels.length === 0) {
    clearGraph();
    return;
  }
  isLoading.value = true;
  statusMessage.value = `Đang tải...`;
  const cypher = `MATCH (n) WHERE any(label IN labels(n) WHERE label IN $labels) OPTIONAL MATCH (n)-[r]-(m) RETURN n, r, m`;
  neovisInstance.renderWithCypher(cypher, { labels });
};

const toggleNodesByType = (label) => {
  if (activeNodeLabels.value.has(label)) activeNodeLabels.value.delete(label);
  else activeNodeLabels.value.add(label);
  renderActiveGraph();
};

const showAllNodes = () => {
  nodeLabelCounts.value.forEach((item) => activeNodeLabels.value.add(item.label));
  renderActiveGraph();
};

const clearGraph = () => {
  stopBlinkingAnimation();
  if (!neovisInstance) return;
  activeNodeLabels.value.clear();
  selectedNodeDetails.value = null;
  neovisInstance.renderWithCypher("MATCH (n) WHERE false RETURN n");
  statusMessage.value = "Đã xóa đồ thị.";
};

// --- Lifecycle Hooks ---
onMounted(async () => {
  await fetchOverviewData();
  await nextTick();
  const dynamicLabelsConfig = {};
  nodeLabelCounts.value.forEach((item) => (dynamicLabelsConfig[item.label] = { color: item.color }));
  const config = {
    containerId: "viz",
    neo4j: { serverUrl: NEO4J_URL, serverUser: NEO4J_USER, serverPassword: NEO4J_PASSWORD },
    initialCypher: `MATCH (n) WHERE false RETURN n`,
    visConfig: {
      physics: {
        enabled: true,
        forceAtlas2Based: { gravitationalConstant: -50, centralGravity: 0.01, springLength: 230, springConstant: 0.05 },
        maxVelocity: 50,
        minVelocity: 0.75,
        solver: "forceAtlas2Based",
        stabilization: { iterations: 2000 },
      },
      interaction: { hover: false },
    },
    labels: {
      ...dynamicLabelsConfig,
      [Neovis.NEOVIS_DEFAULT_CONFIG]: {
        size: 25,
        label: (n) => n.properties?.displayName || n.properties?.entityId || n.properties?.type || "",
      },
    },
  };
  neovisInstance = new Neovis(config);
  neovisInstance.render();
  neovisInstance.registerOnEvent("completed", (eventData) => {
    isLoading.value = false;
    const graph = eventData?.graph;
    const nodeCount = graph?.nodes?.length || 0;
    const edgeCount = graph?.edges?.length || 0;
    if (nodeCount > 0) {
      statusMessage.value = `Hiển thị ${nodeCount} nodes và ${edgeCount} relationships.`;
      setTimeout(() => {
        statusMessage.value = "";
      }, 2000);
    } else {
      statusMessage.value = "Vui lòng chọn một loại node để hiển thị.";
    }
    const network = neovisInstance.network;
    if (network) {
      const blinkingNodeIds = [];
      const visNodes = network.body.data.nodes.get();
      visNodes.forEach((node) => {
        const props = node.raw?.properties;
        if (props) {
          const nodeName = props.displayName || props.name;
          if (alertedEntityNames.value.has(nodeName)) blinkingNodeIds.push(node.id);
        }
      });
      startBlinkingAnimation(blinkingNodeIds);
      network.off("click");
      network.on("click", (params) => {
        if (params.nodes.length > 0) {
          const nodeId = params.nodes[0];
          const visNode = neovisInstance.network.body.data.nodes.get(nodeId);
          const rawNeo4jNode = visNode?.raw;
          if (rawNeo4jNode) {
            const nodeName = rawNeo4jNode.properties?.displayName || rawNeo4jNode.properties?.name;
            const associatedAlerts = alertsByDisplayName.value.get(nodeName);
            selectedNodeDetails.value = { ...rawNeo4jNode, alerts: associatedAlerts || [] };
          } else {
            selectedNodeDetails.value = null;
          }
        } else {
          selectedNodeDetails.value = null;
        }
      });
    }
  });
  neovisInstance.registerOnEvent("error", (e) => {
    isLoading.value = false;
    console.error("Lỗi từ Neovis:", e);
    statusMessage.value = "Đã xảy ra lỗi khi vẽ đồ thị.";
  });
});

onUnmounted(() => {
  stopBlinkingAnimation();
  if (neovisInstance && typeof neovisInstance.network?.destroy === "function") {
    neovisInstance.network.destroy();
  }
  neovisInstance = null;
});
</script>

<style>
/* Đảm bảo layout chiếm toàn bộ màn hình */
html,
body,
#app {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
  background-color: #f9fafb; /* bg-gray-50 */
}
.font-sans {
  font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial,
    "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
}
</style>
