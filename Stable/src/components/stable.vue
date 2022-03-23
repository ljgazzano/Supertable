<template>
  <div>
    <div class="waitAction" v-if="varWaitAction">
      <p>loading process...</p>
    </div>
    <div class="waitAction" style="background-color: blue" v-if="loading">
      <p>loading data...</p>
    </div>
    <div v-if="!cmDisableGroup">
      <label for="grupo">Agrupar</label>
      <select v-model="groupby" id="grupo">
        <option value="">No agrupar</option>
        <option
          v-for="(item, index) in headers"
          :key="index"
          :value="item.value"
        >
          {{ item.text }}
        </option>
      </select>
    </div>

    <div v-if="cmDisableFilter">
      <select name="" id="" v-model="selectedFilter">
        <option
          v-for="(header, index) of headersSelected"
          :key="index"
          :value="header.value"
        >
          {{ header.text }}
        </option>
      </select>
      <label for="filter">Filtro</label>
      <input type="text" id="filter" v-model="filter" />
    </div>

    <div v-if="!cmDisableChangeColumns">
      <a href="#miModal" @click="fnPreventOrder">Abrir Modal</a>
      <div id="miModal" class="modal">
        <div class="modal-contenido">
          <a href="#">X</a>
          <h2>Selector de Columnas</h2>
          <div>
            <table>
              <tr v-for="(header, index) in headers" :key="index">
                <td>
                  <div>
                    <input
                      type="checkbox"
                      :id="header.value"
                      :value="header"
                      v-model="headersSelected"
                    />
                    <label :for="header.value">{{ header.text }}</label>
                  </div>
                </td>
              </tr>
            </table>
          </div>
        </div>
      </div>
    </div>
    <div>
      <div>
        <div v-if="!varListGroupsTabsLimitExceded" class="tabsgrouping">
          <div
            v-for="(head, index) in varListGroupsTabs"
            :key="index"
            class="tabsgroupbtns"
          >
            <button @click="varGroupTabSelected = head" :value="head">
              {{ head }}
            </button>
          </div>
          <br />
        </div>
        <div v-else>
          <p>El número de grupos excede el límite máximo</p>
        </div>
      </div>
      <table>
        <thead v-if="cmDisableHeader">
          <tr>
            <th v-if="letRowSelect">
              <input
                type="checkbox"
                name=""
                id=""
                @click="fnSelectAll()"
                v-model="varIsCheckAll"
              />
            </th>
            <th
              v-for="(header, index) in headersSelected"
              :key="index"
              @click="fnSortHeaderByClick(header.value)"
              v-bind:class="{
                pointer: !cmDisableSort,
                nopointer: cmDisableSort,
              }"
            >
              {{ header.text }}
              <span v-if="!cmDisableSort">
                <img
                  src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOAQMAAAAlhr+SAAAAAXNSR0IArs4c6QAAAAZQTFRFAAAAAAAApWe5zwAAAAF0Uk5TAEDm2GYAAAAfSURBVAjXY2CAAeYGBsYDDAwPGOw/gBCQAeQCBWEAAIj7Bqe8t8xtAAAAAElFTkSuQmCC"
                  alt=""
                  class=""
                  v-bind:class="{
                    iconsortasc: sortasc,
                    iconsortdesc: !sortasc,
                  }"
                  v-if="sortby == header.value"
                />
              </span>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(items, i) in cmTbodydata" :key="i">
            <td v-if="letRowSelect">
              <input
                type="checkbox"
                :value="fnSelectedItemsReturnMode(items)"
                v-model="RowSelected"
              />
            </td>
            <td v-for="(item, x) in items" :key="x">
              <div v-if="extensibleRow"></div>
              <div v-else>
                {{ item }}
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import Tbody from "./tbody.vue";

export default {
  components: { Tbody },
  props: {
    headers: Array,
    items: Array,
    limitGroups: Number,
    disableFilter: Boolean,
    disableSort: Boolean,
    disableHeader: Boolean,
    disableGroup: Boolean,
    disableChangeColumns: Boolean,
    letRowSelect: Boolean,
    RowSelectObject: Boolean,
    loading: Boolean,
    extensibleRow: Boolean,
  },
  data: function () {
    return {
      filter: "",
      selectedFilter: "",
      groupby: null,
      headersSelected: [],
      RowSelected: [],
      sortby: null,
      sortasc: false,
      preventOrder: false,
      arrGroupData: null,
      varListGroupsTabs: null,
      varListGroupsTabsLimitExceded: false,
      varGroupTabSelected: null,
      varIsCheckAll: false,
      varInternalTimer: false,
      varWaitAction: false,
      auxInternalTimer: null,
    };
  },
  computed: {
    cmTbodydata: function () {
      const predata = this.fnFilterData(this.items, this.selectedFilter);
      if (!this.groupby) {
        return this.fnRenderTable(predata);
      } else {
        this.fnRenderGroupTabs();
      }
      if (this.varGroupTabSelected) {
        return this.fnRenderTable(this.arrGroupData[this.varGroupTabSelected]);
      }
    },
    cmLimitGroupsQuantity: function () {
      return this.limitGroups || 10;
    },
    cmDisableFilter: function () {
      return !this.disableFilter;
    },
    cmDisableSort: function () {
      return this.disableSort;
    },
    cmDisableHeader: function () {
      return !this.disableHeader;
    },
    cmDisableGroup: function () {
      return this.disableGroup;
    },
    cmDisableChangeColumns: function () {
      return this.disableChangeColumns;
    },
    cmHeaderSelectable: function () {
      return (
        this.headersSelected.filter((x) => x.select == true)[0].value || false
      );
    },
  },
  methods: {
    fnOrdenInt(a, b) {
      return parseInt(a[this.sortby], 10) - parseInt(b[this.sortby], 10);
    },
    fnOrderString(a, b) {
      if (a[this.sortby] > b[this.sortby]) {
        return 1;
      } else if (a[this.sortby] < b[this.sortby]) {
        return -1;
      }
      return 0;
    },
    fnSortByNumber(sortby) {
      let verification = false;
      this.headers.forEach((x) => {
        if (x.value == sortby) {
          verification = x.type ? (x.type == "number" ? true : false) : false;
        }
      });
      return verification;
    },
    fnSortHeaderByClick(val) {
      if (!this.cmDisableSort) {
        this.preventOrder = false;
        this.sortasc = !this.sortasc;
        this.sortby = val;
      }
    },
    fnPreventOrder() {
      this.preventOrder = true;
    },
    fnTabsGroup(tab) {
      var i;
      var x = document.getElementsByClassName("tabsgroupcontentdata");
      for (i = 0; i < x.length; i++) {
        x[i].style.display = "none";
      }
      document.getElementById(tab).style.display = "block";
    },
    fnGroupData(groupby) {
      const result = this.items.reduce(function (r, a) {
        r[a[groupby]] = r[a[groupby]] || [];
        r[a[groupby]].push(a);
        return r;
      }, Object.create(null));
      return result;
    },
    fnRenderTable(datos) {
      if (!this.preventOrder) {
        if (this.sortby) {
          if (this.fnSortByNumber(this.sortby)) {
            datos.sort(this.fnOrdenInt);
          } else {
            datos.sort(this.fnOrderString);
          }
        }
        if (!this.sortasc) datos = datos.reverse();
      }
      const bodydata = datos.map((x) => {
        let item = {};
        this.headersSelected.forEach((element) => {
          item[element.value] = x[element.value];
        });
        return item;
      });
      return bodydata;
    },
    fnFilterData(items, prop) {
      try {
        const cond = (item) =>
          item[prop].toString().includes(this.filter.toString());
        const result = items.filter(cond);
        return result;
      } catch (error) {
        return items;
      }
    },
    fnRenderGroupTabs() {
      this.arrGroupData = this.fnGroupData(this.groupby);
      const countgroups = Object.keys(this.arrGroupData).length;
      if (countgroups < this.cmLimitGroupsQuantity) {
        this.varListGroupsTabsLimitExceded = false;
        this.varListGroupsTabs = Object.keys(this.arrGroupData);
        return;
      } else {
        this.varListGroupsTabsLimitExceded = true;
      }
    },
    fnSelectedItemsReturnMode(item) {
      return this.RowSelectObject ? item : item[this.cmHeaderSelectable];
    },
    fnSelectAll() {
      if (this.varIsCheckAll) {
        this.RowSelected = this.cmTbodydata.map((x) => {
          return this.RowSelectObject ? x : this.fnSelectedItemsReturnMode(x);
        });
        return;
      }
      this.RowSelected = [];
    },
    fnActiveProcessWait() {
      // in develop
      this.varInternalTimer = !this.varInternalTimer;
    },
  },
  created: async function () {
    this.headersSelected = this.headers;
  },
  watch: {
    RowSelected: function () {
      this.$emit("returnSelectedItems", this.RowSelected);
    },
    varIsCheckAll: function () {
      this.fnSelectAll();
    },
    // varInternalTimer: function (val) {
    //   if (val) {
    //     let count = 0;
    //     this.auxInternalTimer = setInterval(() => {
    //       count += 1;
    //       console.log(this.varInternalTimer, count);
    //       if (count > 1) {
    //         this.varWaitAction = true;
    //         count = 0;
    //         clearInterval(this.auxInternalTimer);
    //       }
    //     }, 1000);
    //   } else {
    //     clearInterval(this.auxInternalTimer);
    //     this.varWaitAction = false;
    //   }
    // },
  },
};
</script>

<style  >
.modal-contenido {
  background-color: aqua;
  width: 300px;
  padding: 10px 20px;
  margin: 20% auto;
  position: relative;
}
.modal {
  background-color: rgba(0, 0, 0, 0.8);
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  opacity: 0;
  pointer-events: none;
  transition: all 1s;
}
#miModal:target {
  opacity: 1;
  pointer-events: auto;
}

.iconsortasc {
  transform: rotate(90deg);
  animation: rotationasc 0.3s;
  animation-fill-mode: forwards;
}

.iconsortdesc {
  transform: rotate(90deg);
  animation: rotationdesc 0.3s;
  animation-fill-mode: forwards;
}

@keyframes rotationdesc {
  from {
    transform: rotate(90deg);
  }
  to {
    transform: rotate(270deg);
  }
}

@keyframes rotationasc {
  from {
    transform: rotate(270deg);
  }
  to {
    transform: rotate(90deg);
  }
}

.pointer {
  cursor: pointer;
}

.nopointer {
  cursor: default;
}
/* tabs */

.tabsgroupbtns {
  float: left;
}

.tabsgroupbtns button {
  max-width: 200px;
  width: 163px;
  height: 33px;
  text-transform: capitalize;
  background-color: transparent;
  border-left: 1px solid black;
  border-right: 1px solid black;
  border-top: none;
  border-bottom: none;
}

.tabsgrouping {
  width: 100%;
  margin: auto;

  display: flex;
}

.tabsgroupbtns button:hover {
  animation: tabselect 0.2s;
  animation-fill-mode: forwards;
}

.tabsgroupbtns button:active {
  background: #c1bbca;
}

@keyframes tabselect {
  from {
    text-decoration: none;
  }
  to {
    text-decoration: underline;
  }
}

.tabsgroupcontentdata {
  display: none;
}

.waitAction {
  width: 100%;
  height: 100%;
  background-color: red;
  margin: auto;
  display: flex;
  z-index: 999;
}
</style>