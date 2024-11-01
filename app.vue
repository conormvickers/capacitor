<template>
  <div class="card" style="width: 100%; height: 100%">
    <div
      id="mynetwork"
      class="card-body"
      style="
        z-index: 1;
        width: 100%;
        height: 100%;
        /* position: absolute; */
        top: 0;
        left: 0;
      "
    ></div>
    <div
      class="pa-6"
      style="bottom: 0; right: 0; position: absolute; margin: 0 0 16px 16px"
    ></div>
  </div>
  <v-dialog v-model="showDeleted">
    <v-card>
      <v-card-title> Deleted Nodes </v-card-title>
      <v-card-text>
        <v-list>
          <v-list-item
            v-for="(item, index) in deletedNodes"
            :key="index"
            :title="item ? (item.label ? item.label : '?') : '?'"
            @click="restoreNode(item)"
          >
          </v-list-item>
        </v-list>
      </v-card-text>
      <v-card-actions>
        <v-btn @click="purgeDeleted()">Purge Deleted</v-btn>
        <v-btn @click="showDeleted = false">Close</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  <v-bottom-sheet
    v-model="showtodo"
    @click:outside="onSheetDismissed"
    max-height="60vh"
  >
    <v-card
      ><v-card-title> To Do </v-card-title
      ><v-card-text>
        <v-list>
          <v-container>
            <v-row v-for="todo in todolist">
              <v-col>
                <v-row align="center" v-for="task in todo.tasks"
                  ><v-checkbox
                    class="nopadding-checkbox"
                    hide-details
                    v-model="task.done"
                  ></v-checkbox
                  ><span>{{ task.task }}</span></v-row
                >
              </v-col>
              <v-col>
                <v-chip @click="zoomTo(todo.goal)"> {{ todo.goal }} </v-chip
                ><v-icon @click="addAndSelect(todo.goal)">mdi-plus</v-icon>
              </v-col>

              <v-divider></v-divider> </v-row
          ></v-container> </v-list
      ></v-card-text>
    </v-card>
  </v-bottom-sheet>

  <v-dialog
    v-model="menu"
    width="1024"
    @click:outside="closeMenu"
    v-on:keydown.enter="menu = false"
  >
    <v-card min-width=" 300" v-click-outside="handleDismiss">
      <v-list>
        <v-list-item :title="selectedNodeName" :subtitle="currentType">
          <v-btn
            variant="text"
            :class="fire ? 'text-red' : ''"
            icon="mdi-fire"
            @click="
              fire = !fire;
              frozen = false;
              pinned = false;
              updateNode();
            "
          ></v-btn>
          <v-btn
            variant="text"
            :class="frozen ? 'text-red' : ''"
            icon="mdi-snowflake"
            @click="
              frozen = !frozen;
              pinned = false;
              fire = false;
              updateNode();
            "
          ></v-btn>
          <v-btn
            variant="text"
            :class="pinned ? 'text-red' : ''"
            icon="mdi-pin"
            @click="
              pinned = !pinned;
              fire = false;
              frozen = false;
              updateNode();
            "
          ></v-btn>
        </v-list-item>
      </v-list>
      <v-text-field
        autofocus
        @focusout="updateNode"
        @focus="$event.target.select()"
        v-model="selectedNodeName"
        label="Item"
      ></v-text-field>
      <v-divider></v-divider>
      <v-text-field
        @focusout="updateNode"
        @focus="$event.target.select()"
        v-model="nodeDetails"
        @input="updateNode"
        label="Details"
      ></v-text-field>
      <v-row align="center" justify="center">
        <v-btn icon="mdi-open-in-new" v-if="isValidUrl" @click="openLink">
        </v-btn
      ></v-row>
      <v-row align="center" justify="center">
        <v-col cols="auto">
          <v-btn color="indigo-lighten-4" @click="addAttachedNode"
            >Add Step</v-btn
          ></v-col
        >
        <v-col cols="auto">
          <v-btn color="indigo-lighten-4" @click="insertNode"
            >Insert Step</v-btn
          ></v-col
        >

        <v-col cols="auto">
          <v-btn
            color="indigo-darken-1"
            @click="
              connectNext = true;
              menu = false;
            "
            >Connect</v-btn
          ></v-col
        >
      </v-row>

      <v-card-actions>
        <v-spacer></v-spacer>

        <v-btn color="red" variant="outlined" @click="deleteNode()"
          >Delete</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
  <v-dialog v-model="edgeMenu" width="1024">
    <v-card min-width=" 300" v-click-outside="handleDismiss">
      <v-row align="center" justify="center">
        <v-col>
          <v-list-item :title="selectedEdgeTo"> </v-list-item>
          <v-list-item :title="selectedEdgeFrom"> </v-list-item
        ></v-col>
        <v-col
          ><v-btn icon="mdi-swap-vertical" @click="flipEdge"> </v-btn
        ></v-col>
      </v-row>

      <v-card-actions>
        <v-spacer></v-spacer>

        <v-btn color="red" variant="outlined" @click="deleteEdge()"
          >Delete</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
  <v-snackbar
    location="top"
    :color="saveStatus"
    v-model="snackbar"
    :timeout="timeout"
  >
    {{ snackText }}

    <template v-slot:actions>
      <v-btn variant="text" @click="snackbar = false"> Close </v-btn>
    </template>
  </v-snackbar>
  <div class="editArea">
    <div>
      <v-row>
        <v-btn icon="mdi-checkbox-marked" @click="showtodo = !showtodo"></v-btn>
        <v-btn icon="mdi-recycle" @click="showDeleted = !showDeleted"></v-btn>
        <v-btn icon="mdi-content-save" @click="save_changes"></v-btn>
        <v-btn icon="mdi-magnify-minus-outline" @click="zoomOut"></v-btn>
        <v-btn icon="mdi-plus" @click="addFreeNode"></v-btn>

        <v-btn icon="mdi-clipboard" @click="clipboard = true"></v-btn>
        <v-menu activator="#menu-activator">
          <v-list>
            <v-list-item
              v-for="(item, index) in userNetworks"
              :key="index"
              :value="index"
            >
              <v-list-item-title @click="getDiagram(item.title)">{{
                item.title
              }}</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </v-row>
    </div>
  </div>
  <v-dialog v-model="clipboard" type="info" dense text max-width="500">
    <v-card class="pa-4">
      <v-form>
        <v-text-field v-model="field1" label="Nodes"></v-text-field>
        <v-text-field v-model="field2" label="Edges"></v-text-field>
      </v-form>
      <v-card-actions
        ><v-btn @click="loadClipbaord()">Load</v-btn></v-card-actions
      >
    </v-card>
  </v-dialog>
</template>

<script setup>
import { Network, DataSet } from "vis-network/standalone";

var nodejson = ref({});
var field1 = "";
var field2 = "";
var clipboard = false;
var nodetable = [
  { id: 1, label: "John Doe", shape: "dot" },
  { id: 2, label: "Jane Smith", shape: "dot" },
  { id: 3, label: "Bob Johnson", shape: "dot" },
];
var edgesTable = [
  { id: 1, from: "1", to: "1" },
  { id: 2, from: "1", to: "2" },
  { id: 3, from: "1", to: "3" },
];
var selectedNode = ref("");
var selectedNodeName = ref("");

var currentType = "";

var fire = ref(false);
var frozen = ref(false);
var pinned = ref(false);
var nodeDetails = ref("");
var menu = ref(false);
var isValidUrl = ref(false);
var connectNext = false;

var showtodo = ref(false);
var todolist = ref([]);
var showDeleted = ref(false);

var edgeMenu = ref(false);
var selectedEdge = ref("");
var selectedEdgeFrom = "";
var selectedEdgeTo = "";
var fireColors = [
  "ff7b00",
  "ff8800",
  "ff9500",
  "ffa200",
  "ffaa00",
  "ffb700",
  "ffc300",
  "ffd000",
  "ffdd00",
  "ffea00",
];
var frozenColors = [
  "73C9FF",
  "7FCAFD",
  "8ACAFA",
  "96CBF8",
  "A2CBF5",
  "ADCCF3",
  "B9CCF0",
];

function zoomTo(title) {
  const findnode = nodes.get({
    filter: function (item) {
      return item.label == title;
    },
  })[0];
  console.log(findnode);

  if (findnode) {
    showtodo.value = false;
    network.fit({ nodes: [findnode.id] });
  }
}

function addAndSelect(title) {
  console.log("searching for ", title);
  const findnode = nodes.get({
    filter: function (item) {
      return item.label == title;
    },
  })[0];

  if (findnode) {
    console.log("found", findnode);
    selectedNode.value = findnode.id;
    showtodo.value = false;
    addAttachedNode();
  } else {
    console.log("ERROR finding node");
  }
}

function onSheetDismissed() {
  var toremove = [];
  for (var i = 0; i < todolist.value.length; i++) {
    for (var j = 0; j < todolist.value[i].tasks.length; j++) {
      if (todolist.value[i].tasks[j].done) {
        toremove.push(todolist.value[i].tasks[j]);
        todolist.value[i].tasks.splice(j, 1);
      }
    }
  }

  for (var i = 0; i < toremove.length; i++) {
    const nodetoremove = nodes.get(toremove[i].id);
    deleteNode(nodetoremove);
  }
}

function restoreNode(node) {
  for (var i = 0; i < node.edges.length; i++) {
    edges.update(node.edges[i]);
  }

  nodes.update(node);

  const index = deletedNodes.value.findIndex((object) => object.id === node.id);
  deletedNodes.value.splice(index, 1);

  save_changes();
}

function getToDo() {
  const firenodes = nodes.get({
    filter: function (node) {
      return node.type == "fire";
    },
  });

  var todo = [];
  for (var i = 0; i < firenodes.length; i++) {
    const pinned = findPinnedPoint(firenodes[i].id)[0];

    var found = false;

    for (var j = 0; j < todo.length; j++) {
      if (todo[j].goal == pinned) {
        todo[j].tasks.push({
          task: firenodes[i].label,
          id: firenodes[i].id,
          done: false,
        });
        found = true;
      }
    }

    if (!found) {
      todo.push({
        tasks: [{ task: firenodes[i].label, id: firenodes[i].id, done: false }],
        goal: pinned,
      });
    }
  }

  todolist.value = todo;
}

function getEdgesTo(nodeId) {
  var connectedEdges = edges.get({
    filter: function (edge) {
      return edge.to == nodeId;
    },
  });

  return connectedEdges;
}

function getEdgesFrom(nodeId) {
  var connectedEdges = edges.get({
    filter: function (edge) {
      return edge.from == nodeId;
    },
  });

  return connectedEdges;
}

function findPinnedPoint(nodeId, seenEdges = []) {
  var connectedEdges = getEdgesFrom(nodeId);

  var nodeends = [];

  if (connectedEdges.length > 0) {
    for (var i = 0; i < connectedEdges.length; i++) {
      if (!seenEdges.includes(connectedEdges[i])) {
        const tonode = nodes.get(connectedEdges[i].to);

        if (tonode && tonode.type == "pinned") {
          nodeends.push(tonode.label);
        } else {
        }

        seenEdges.push(connectedEdges[i]);
        if (getEdgesFrom(connectedEdges[i].to).length > 0) {
          const toadd = findPinnedPoint(connectedEdges[i].to, seenEdges);
          nodeends = nodeends.concat(toadd);
        }
      }
    }
  }

  return nodeends;
}

function findEndPoint(nodeId, seenEdges = []) {
  var connectedEdges = getEdgesFrom(nodeId);

  var nodeends = [];

  if (connectedEdges.length > 0) {
    for (var i = 0; i < connectedEdges.length; i++) {
      if (!seenEdges.includes(connectedEdges[i])) {
        nodeends.push(nodes.get(connectedEdges[i].to).label);
        seenEdges.push(connectedEdges[i]);
        if (getEdgesFrom(connectedEdges[i].to).length > 0) {
          const toadd = findEndPoint(connectedEdges[i].to, seenEdges);
          nodeends = nodeends.concat(toadd);
        }
      }
    }
  }

  return nodeends;
}

var pinnedColors = ["fdc5f5", "f7aef8", "b388eb", "8093f1", "72ddf7"];
var normalColors = ["E5E5EA", "C9C9CE ", "A5A5A8 "];

var nodes = new DataSet(nodetable);
var edges = new DataSet(edgesTable);
var deletedNodes = ref([]);

var network = null;

var snackbar = false;
var snackText = "";
var saveStatus = "success";
var timeout = 2000;

function handleDismiss(event) {
  menu.value = false;
  network.unselectAll();
  if (!event.target.classList.contains("v-btn")) {
    updateNode();
  }
}

function isValid() {
  if (!nodeDetails.value) return false;

  return validateUrl(nodeDetails.value);
}

function addEdge() {
  // Add an Edge to the array
  edgesTable.push({ id: Edges.length + 1, to: "", from: "" });
}
function updateEdge(Edge, index) {
  // Update the Edge in the array
  edgesTable.splice(index, 1, Edge);
  updateNetwork();
}
function deleteEdge() {
  edges.remove(selectedEdge.value);
  edgeMenu.value = false;
  save_changes();
}

function purgeDeleted() {
  deletedNodes.value = [];
  save_changes();
  showDeleted.value = false;
}
function deleteNode(todelete = selectedNode.value) {
  if (typeof todelete == "string") {
    todelete = nodes.get(todelete);
  }
  var connectedEdges = [];
  getEdgesFrom(todelete).forEach((edge) => {
    connectedEdges.push(edge);
    edges.remove(edge);
  });
  getEdgesTo(todelete).forEach((edge) => {
    connectedEdges.push(edge);
    edges.remove(edge);
  });
  todelete.edges = connectedEdges;

  deletedNodes.value.push(todelete);

  nodes.remove(todelete);
  menu.value = false;
  save_changes();
}
function addItem() {
  // Add an item to the array
  nodetable.push({ id: nodetable.length + 1, label: "New Item" });
}
function updateItem(item, index) {
  // Update the item in the array
  nodetable.splice(index, 1, item);
  updateNetwork();
}
function deleteItem(index) {
  // Remove the item from the array
  nodetable.splice(index, 1);
}
function zoomOut() {
  network.unselectAll();
  network.fit({ animation: { duration: 300 } });
}

async function save_changes() {
  const nodestosave = JSON.stringify(nodes.get()); // nodes.get();
  const edgestosave = JSON.stringify(edges.get()); // edges.get();
  const deletedNodesjson = JSON.stringify(deletedNodes.value);

  const result = await fetch("https://dev.docdrive.link/api/savenetwork", {
    method: "POST",
    body: JSON.stringify({
      data: {
        nodes: nodestosave,
        edges: edgestosave,
        deletedNodes: deletedNodesjson,
      },
    }),
  });
  console.log(result);

  getToDo();
}

function openLink() {
  window.open(nodeDetails.value, "_blank", "noreferrer");
}
function validateUrl(value) {
  if (!value) return false;
  value = value.trim();
  return /^(?:(?:(?:https?|ftp):)?\/\/)(?:\S+(?::\S*)?@)?(?:(?!(?:10|127)(?:\.\d{1,3}){3})(?!(?:169\.254|192\.168)(?:\.\d{1,3}){2})(?!172\.(?:1[6-9]|2\d|3[0-1])(?:\.\d{1,3}){2})(?:[1-9]\d?|1\d\d|2[01]\d|22[0-3])(?:\.(?:1?\d{1,2}|2[0-4]\d|25[0-5])){2}(?:\.(?:[1-9]\d?|1\d\d|2[0-4]\d|25[0-4]))|(?:(?:[a-z\u00a1-\uffff0-9]-*)*[a-z\u00a1-\uffff0-9]+)(?:\.(?:[a-z\u00a1-\uffff0-9]-*)*[a-z\u00a1-\uffff0-9]+)*(?:\.(?:[a-z\u00a1-\uffff]{2,})))(?::\d{2,5})?(?:[/?#]\S*)?$/i.test(
    value
  );
}

function showMenu(input) {
  if (input["nodes"] && input["nodes"].length > 0) {
    if (input["nodes"][0].toString().startsWith("cluster")) {
      network.openCluster(input["nodes"][0]);
    } else {
      if (connectNext) {
        edges.add({
          from: selectedNode.value,
          to: input["nodes"][0],
          arrows: {
            from: {
              enabled: false,
            },
            to: {
              enabled: true,

              type: "arrow",
            },
          },
        });
        connectNext = false;
      } else {
        menu.value = true;
        const nodeRef = nodes.get(input["nodes"][0]);
        nodejson.value = nodeRef;
        selectedNode.value = nodeRef.id;
        selectedNodeName.value = nodeRef.label;
        nodeDetails.value = nodeRef.details;
        currentType = nodeRef.type || "";
        fire.value = currentType == "fire";
        frozen.value = currentType == "frozen";
        pinned.value = currentType == "pinned";
        isValidUrl.value = validateUrl(nodeDetails.value);
      }
    }
  } else if (
    input["edges"] &&
    input["edges"].length > 0 &&
    menu.value == false
  ) {
    selectedEdge.value = input["edges"][0];
    selectedEdgeFrom = selectedEdge.from;
    selectedEdgeTo = selectedEdge.to;
    edgeMenu.value = true;
  } else {
    menu.value = false;
    edgeMenu.value = false;
  }
}
async function updateNode() {
  var nodeRef = nodes.get(selectedNode.value);
  nodeRef["label"] = selectedNodeName.value;
  if (fire.value) {
    nodeRef["type"] = "fire";
    nodeRef["color"] =
      "#" + fireColors[Math.floor(Math.random() * fireColors.length)];
    nodeRef["shape"] = "circle";
  } else if (frozen.value) {
    nodeRef["type"] = "frozen";
    nodeRef["color"] =
      "#" + frozenColors[Math.floor(Math.random() * frozenColors.length)];
    nodeRef["shape"] = "dot";
    nodeRef["size"] = 12;
  } else if (pinned.value) {
    nodeRef["type"] = "pinned";
    nodeRef["color"] =
      "#" + pinnedColors[Math.floor(Math.random() * pinnedColors.length)];
    nodeRef["shape"] = "circle";
  } else {
    nodeRef["type"] = "normal";
    nodeRef["color"] =
      "#" + normalColors[Math.floor(Math.random() * normalColors.length)];
    nodeRef["shape"] = "dot";
    nodeRef["size"] = 20;
  }

  nodeRef["details"] = nodeDetails.value;

  isValidUrl.value = await validateUrl(nodeDetails.value);

  await nodes.update(nodeRef);

  save_changes();
}

function addFreeNode() {
  nodes.update({ label: "NEW", shape: "dot" });
  const newNode = nodes.get({
    filter: function (item) {
      return item.label == "NEW";
    },
  });
  showMenu({ nodes: [newNode[0].id] });
}

function addAttachedNode() {
  nodes.update({ label: "NEW", shape: "dot" });
  const newNode = nodes.get({
    filter: function (item) {
      return item.label == "NEW";
    },
  });

  edges.add({
    from: newNode[0].id,
    to: selectedNode.value,
    arrows: {
      from: {
        enabled: false,
      },
      to: {
        enabled: true,

        type: "arrow",
      },
    },
  });
  showMenu({ nodes: [newNode[0].id] });
}

function insertNode() {
  nodes.update({ label: "NEW", shape: "dot" });
  const newNode = nodes.get({
    filter: function (item) {
      return item.label == "NEW";
    },
  })[0];

  const insertedge = edges.get({
    filter: function (item) {
      return item.from == selectedNode.value;
    },
  })[0];

  edges.add({
    from: selectedNode.value,
    to: newNode.id,
    arrows: {
      from: {
        enabled: false,
      },
      to: {
        enabled: true,

        type: "arrow",
      },
    },
  });
  edges.add({
    from: newNode.id,
    to: insertedge.to,
    arrows: {
      from: {
        enabled: false,
      },
      to: {
        enabled: true,

        type: "arrow",
      },
    },
  });

  edges.remove(insertedge);

  showMenu({ nodes: [newNode.id] });
}

function flipEdge() {
  const edgetoflip = edges.get(selectedEdge.value);
  const oldfrom = edgetoflip.from;
  const oldto = edgetoflip.to;

  edges.update({
    from: oldto,
    to: oldfrom,
    id: edgetoflip.id,
    arrows: {
      to: {
        enabled: true,

        type: "arrow",
      },
      from: {
        enabled: false,
      },
    },
  });
  save_changes();
}

async function loadNetwork() {
  const response = await fetch("https://dev.docdrive.link/api/getnetwork", {});
  const data = await response.json();
  console.log(data);
  if (data.network) {
    nodetable = JSON.parse(data.network.nodes);

    edgesTable = JSON.parse(data.network.edges);
    deletedNodes.value = JSON.parse(data.network.deleted ?? "[]");
  }
  updateNetwork();
}
function loadClipbaord() {
  clipboard = false;
  nodetable = JSON.parse(field1.replaceAll("\\", ""));

  edgesTable = JSON.parse(field2.replaceAll("\\", ""));

  updateNetwork();
}
function updateNetwork() {
  var container = document.getElementById("mynetwork");

  nodes = new DataSet(nodetable);
  nodes.forEach((nodeRef) => {
    nodeRef.font = "";
  });

  const ids = nodetable.map((item) => item.id);

  edgesTable = edgesTable.filter(
    (edge) => ids.includes(edge.to) && ids.includes(edge.from)
  );
  edges = new DataSet(edgesTable);

  var data = {
    nodes: nodes,
    edges: edges,
  };
  var options = {
    layout: {
      // hierarchical: {
      //   direction: "UD",
      //   sortMethod: "hubsize",
      //   treeSpacing: 10,
      // },
    },
  };
  network = new Network(container, data, options);
  network.on("selectNode", (event) => {
    const nodeId = event.nodes[0];
    showMenu({ nodes: [nodeId] });
  });
  network.on("selectEdge", (event) => {
    if (!menu.value) {
      const edgeId = event.edges[0];
      showMenu({ edges: [edgeId] });
    }
  });

  getToDo();
}

function closeMenu() {
  network.unselectAll();
}
onMounted(() => {
  // create an array with nodes
  nextTick(async () => {
    await loadNetwork();
  });
});
</script>

<style>
#__nuxt {
  height: 100%;
  width: 100%;
}

div.vis-network {
  background-color: rgba(0, 0, 0, 0);
  height: 100%;
  width: 100%;
}

.editArea {
  position: absolute;
  bottom: 0;
  right: 0;
  height: 200px;
  width: 100px;
  background: #ffffff00;
}
</style>
