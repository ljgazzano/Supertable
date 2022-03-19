<template>
  <div>
    <label for="orden">Orden</label>
    <select v-model="sortby" id="orden">
      <option value="">No ordenar</option>
      <option v-for="(item, index) in headers" :key="index" :value="item.value">
        {{ item.text }}
      </option>
    </select>
    <label for="grupo">Agrupar</label>
    <select v-model="groupby" id="grupo">
      <option value="">No agrupar</option>
      <option v-for="(item, index) in headers" :key="index" :value="item.value">
        {{ item.text }}
      </option>
    </select>

    <div v-if="cmDisableFilter">
      <label for="filter">Filtro</label>
      <input type="text" id="filter" v-model="filter" />
    </div>
    <div>
      <input type="colapsarbtn" value="Columnas" />
    </div>
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

          <!-- <div
            v-for="(head, index) in varListGroupsTabs"
            :key="index"
            class="tabsgroupcontent"
          >
            <div :id="head" class="tabsgroupcontentdata">
              <h2>{{ head }}</h2>
              <p>Solapa de {{ head }}.</p>
            </div>
          </div> -->
        </div>
        <div v-else>
          <p>El número de grupos excede el límite máximo</p>
        </div>
      </div>
      <table>
        <thead v-if="cmDisableHeader">
          <tr>
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
        <Tbody
          :html="cmTbodydata"
          v-if="cmDisableOptimized"
          style="color: green"
        ></Tbody>
        <tr
          v-else
          v-for="(item, index) in cmTbodydata"
          :key="index"
          v-html="item"
          style="color: red"
        ></tr>
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
    disableOptimized: Boolean,
    disableFilter: Boolean,
    disableSort: Boolean,
    disableHeader: Boolean,
  },
  data: function () {
    return {
      filter: "",
      groupby: null,
      headersSelected: [],
      sortby: null,
      sortasc: false,
      preventOrder: false,
      arrGroupData: null,
      varListGroupsTabs: null,
      varListGroupsTabsLimitExceded: false,
      varGroupTabSelected: null,
    };
  },
  computed: {
    cmTbodydata: function () {
      if (!this.groupby) {
        return this.fnRenderTable(this.items);
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
    cmDisableOptimized: function () {
      return !this.disableOptimized;
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
      const countHeaders = this.headersSelected.length;
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

      let bodydata = [];

      datos.forEach((x, index) => {
        let item = "";
        this.headersSelected.forEach((element) => {
          item += `<td>${x[element.value]}</td>`;
        });

        if (this.cmDisableOptimized) {
          if (this.filter) {
            if (item.includes(this.filter)) bodydata += "<tr>" + item + "</tr>";
          } else {
            bodydata += "<tr>" + item + "</tr>";
          }
        } else {
          if (this.filter) {
            if (item.includes(this.filter)) bodydata.push(item);
          } else {
            bodydata.push(item);
          }
        }
      });
      return bodydata;
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
  },
  created: async function () {
    console.log(this.cmLimitGroupsQuantity);
    this.headersSelected = this.headers;
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

.tabsgrouping {
}
.tabsgroupbtns {
  float: left;
}

.tabsgroupbtns button {
  max-width: 200px;
  width: 163px;
  background: #383434;
  color: white;
  height: 33px;
  border-top-left-radius: 12px;
  border-top-right-radius: 12px;
  text-transform: capitalize;
  border: none;
}

.tabsgroupbtns button:hover {
  background: #929191;
  color: white;
  font-size: 105%;
  animation: tabselect 1s;
  animation-fill-mode: forwards;
}

.tabsgroupbtns button:active {
  background: #a55555;
  color: white;
  font-size: 105%;
  animation: tabselected;
  animation-fill-mode: forwards;
}

@keyframes tabselect {
  from {
    transform: scale(100%);
  }
  to {
    transform: scale(110%);
  }
}

@keyframes tabselected {
  from {
    background-color: #383434;
  }
  to {
    background-color: red;
  }
}

.tabsgroupcontent {
}
.tabsgroupcontentdata {
  display: none;
}
</style>