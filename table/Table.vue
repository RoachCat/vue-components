<template>
  <div class="table-component">
    <div class="button-toolbar" v-if="addRows">
      <button class="button-toolbar__button" @click="addRow">Add row</button>
    </div>
    <div class="table-container">
      <table
        v-if="type === 'horizontal'"
        class="table"
        ref="table-component"
        border="1"
        cellspacing="0"
        cellpadding="0"
        :style="
          tableStyles
            ? tableStyles.table
            : 'border-collapse: collapse; border: 1px solid #e6e6fa'
        "
      >
        <thead>
          <slot name="table-header"></slot>
          <tr name="table-fields" v-if="tableFields.length">
            <th
              v-for="(field, index) in tableFields"
              :style="tableStyles ? tableStyles.th : ''"
              :key="'th-' + index"
            >
              {{ field }}
            </th>
            <th
              :style="tableStyles ? tableStyles.th : ''"
              v-if="deleteRows && tableData.length > 1"
            ></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in tableData" :key="'row-' + row.tableRowId">
            <template v-for="(cell, index) in Object.keys(row)">
              <td
                :style="tableStyles ? tableStyles.td : ''"
                v-if="cell !== 'tableRowId' && cell !== 'inputType'"
                :key="'cell-' + index"
                :name="cell"
              >
                <slot :name="cell" :row="row">
                  <span>{{ row[cell] }}</span>
                </slot>
              </td>
            </template>
            <td
              class="table__delete-button"
              v-if="deleteRows && tableData.length > 1"
              name="delete-button"
            >
              <button @click="deleteRow(index)">X</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: "HorizontalTable",
  props: {
    type: {
      type: String,
      default: "horizontal",
    },
    fields: {
      type: Array,
      default: () => [],
    },
    data: {
      type: Array,
      default: () => [],
    },
    addRows: {
      type: Boolean,
      default: false,
    },
    deleteRows: {
      type: Boolean,
      default: false,
    },
    tableStyles: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      tableFields: [],
      tableData: [],
    };
  },
  watch: {
    tableData: {
      deep: true,
      handler() {
        this.$emit("input", this.tableData);
      },
    },
  },
  created() {
    this.tableFields = [...this.fields];
    this.tableData = this.data.map((el) => {
      return {
        ...el,
        tableRowId: this.createGUID(),
      };
    });
  },
  methods: {
    addRow() {
      const newData = {};
      Object.keys(this.tableData[0]).forEach((key) => {
        if (key === "tableRowId") {
          newData[key] = this.createGUID();
        } else {
          newData[key] = "";
        }
      });
      this.tableData.push(newData);
    },
    deleteRow(index) {
      this.tableData.splice(index, 1);
    },
    getTableHtml() {
      const table = this.$refs["table-component"].cloneNode(true);
      // Removing the last column of the table.
      const tableHeader = table.querySelector("thead");
      const deleteButtonCol = tableHeader.lastChild.lastChild;
      if (deleteButtonCol) {
        tableHeader.lastChild.removeChild(deleteButtonCol);
      }
      const tableBody = table.querySelector("tbody");
      // Iterates over each table row, put input values inside span labels and replaces it.
      const rows = tableBody.querySelectorAll("tr");
      rows.forEach((row) => {
        row.childNodes.forEach((cell) => {
          if (cell.nodeName.toLowerCase() === "td") {
            if (cell.getAttribute("name") === "delete-button") {
              row.removeChild(cell);
            } else {
              if (
                cell.childNodes[0].getAttribute("cell-type") === "editable-cell"
              ) {
                const spanLabel = document.createElement("span");
                spanLabel.textContent = cell.childNodes[0].value;
                cell.innerHTML = "";
                cell.appendChild(spanLabel);
              }
            }
          }
        });
      });
      return table;
    },
    createGUID() {
      return "_" + Math.random().toString(36).substr(2, 9);
    },
  },
};
</script>

<style scoped>
.table th,
.table td {
  border: 1px solid black;
  border-collapse: collapse;
}

.table__delete-button {
  text-align: center;
  border: none !important;
}
.table__delete-button button {
  border: none;
  background: none;
  color: red;
}
.button-toolbar {
  margin: 20px 0px;
}
.button-toolbar__button {
  margin: 15px 0px 5px 0px;
  border: 1px solid #b4b4b4;
  border-radius: 8px;
  padding: 8px 8px;
}
</style>
