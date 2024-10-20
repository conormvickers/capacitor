<template>
  <ClientOnly fallback-tag="span" fallback="Loading...">
    <Grid :tableData="tableData" :key="tableDataKey" @update="handleup" />
  </ClientOnly>

  <div id="main" style="width: 600px; height: 400px"></div>
</template>
<script setup>
import * as echarts from "echarts";

function handleup(d) {
  console.log(d);
  console.log(tableData.value);
}

var tableData = ref([
  ["10.26", null, "Sum", "=SUM(A:A)"],
  ["20.12", null, "Average", "=AVERAGE(A:A)"],
  // ...
]);
var tableDataKey = ref(0);

onMounted(() => {
  var myChart = echarts.init(document.getElementById("main"));
  var option = {
    title: {
      text: "ECharts Getting Started Example",
    },
    tooltip: {},
    legend: {
      data: ["sales"],
    },
    xAxis: {
      data: ["Shirts", "Cardigans", "Chiffons", "Pants", "Heels", "Socks"],
    },
    yAxis: {},
    series: [
      {
        name: "sales",
        type: "bar",
        data: [5, 20, 36, 10, 10, 20],
      },
    ],
  };
  myChart.setOption(option);
  function doSomething() {
    // Code to be executed after waiting for 3 seconds
    console.log("Doing something...");
    tableData.value = [["10.adfasdfasdfasdf", null, "Sum", "=SUM(A:A)"]];
    tableDataKey.value++; // increment the key to trigger a reload
  }

  setTimeout(doSomething, 3000);
});
</script>
