<template>
  <div>
    <h3 class="demo-preview">Sheet 1</h3>
    <div id="example-basic-multi-sheet-1"></div>
  </div>
</template>
<script setup>
import Handsontable from "handsontable";
import { HyperFormula } from "hyperformula";
import "handsontable/dist/handsontable.full.min.css";

const props = defineProps({
  // Add your props here, for example:
  tableData: {
    type: Array,
    default: () => [],
  },
  sheetName: {
    type: String,
    default: "Sheet1",
  },
});

const hyperformulaInstance = HyperFormula.buildEmpty({
  // to use an external HyperFormula instance,
  // initialize it with the `'internal-use-in-handsontable'` license key
  licenseKey: "internal-use-in-handsontable",
});

const emit = defineEmits(["update"]);

emit("update", "khgfj");

onMounted(() => {
  console.log(props.tableData);
  const container1 = document.querySelector("#example-basic-multi-sheet-1");
  const table = new Handsontable(container1, {
    data: props.tableData,

    height: "auto",
    formulas: {
      engine: hyperformulaInstance,
      sheetName: "Sheet1",
    },
    afterChange(change, source) {
      if (source === "loadData") {
        return; // don't save this change
      }

      if (!change) {
        return;
      }

      emit("update", table.getData());
      emit("update", props.tableData);
    },
    autoWrapRow: true,
    autoWrapCol: true,
    licenseKey: "non-commercial-and-evaluation",
  });
});
</script>
