<template>
  <div>
    <input type="button" value="Agrupar" @click="fnRenderGroupTabs()" />
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
    <div>
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

    <table v-if="!groupby">
      <thead>
        <tr>
          <th
            v-for="(header, index) in headersSelected"
            :key="index"
            @click="fnSortHeaderByClick(header.value)"
          >
            {{ header.text }}
            <img
              src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOAQMAAAAlhr+SAAAAAXNSR0IArs4c6QAAAAZQTFRFAAAAAAAApWe5zwAAAAF0Uk5TAEDm2GYAAAAfSURBVAjXY2CAAeYGBsYDDAwPGOw/gBCQAeQCBWEAAIj7Bqe8t8xtAAAAAElFTkSuQmCC"
              alt=""
              class=""
              v-bind:class="{ iconsortasc: sortasc, iconsortdesc: !sortasc }"
              v-if="sortby == header.value"
            />
          </th>
        </tr>
      </thead>
      <Tbody :html="cmTbodydata" v-if="optimized" style="color: green"></Tbody>
      <tr
        v-else
        v-for="(item, index) in cmTbodydata"
        :key="index"
        v-html="item"
        style="color: red"
      ></tr>
    </table>

    <div v-else>
      <div v-if="!varListGroupsTabsLimitExceded" class="tabsgrouping">
        <div v-for="(head, index) in varListGroupsTabs" :key="index">
          <button class="" @click="fnTabsGroup(head)">{{ head }}</button>
        </div>
        <div v-for="(head, index) in varListGroupsTabs" :key="index">
          <div :id="head" class="grouptabbtn">
            <h2>{{ head }}</h2>
            <p>Solapa de {{ head }}.</p>
          </div>
        </div>
      </div>
      <div v-else>
        <p>El número de grupos excede el límite máximo</p>
      </div>
    </div>
  </div>
</template>

<script>
import jsonmucho from "../assets/generated.json";
import jsonpoco from "../assets/jsontest";
import Tbody from "./tbody.vue";

export default {
  components: { Tbody },
  data: function () {
    return {
      headers: [
        { value: "_id", text: "ID" },
        { value: "index", text: "indice", type: "number" },
        { value: "guid", text: "guid" },
        { value: "isActive", text: "activo" },
        { value: "balance", text: "balance" },
      ],
      datos: jsonpoco.data(),
      filter: "",
      groupby: "isActive",
      headersSelected: [],
      sortby: null,
      sortasc: false,
      preventOrder: false,
      optimized: false,
      limitGroupsQuantity: 20,
      varListGroupsTabs: null,
      varListGroupsTabsLimitExceded: false,
    };
  },
  computed: {
    cmTbodydata: function () {
      return this.fnRenderTable(this.datos);
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
      this.preventOrder = false;
      this.sortasc = !this.sortasc;
      this.sortby = val;
    },
    fnPreventOrder() {
      this.preventOrder = true;
    },
    fnRequireGroup(groupref, trdata, colspan, index) {
      if (this.groupby) {
        const gruppedhtml = `<tr><td colspan="${colspan}" class="tdgroupcontainer"><input type="checkbox" id="title${index}" /><label for="title${index}">Grupo ${groupref}</label><div class="content"><table>${trdata}</table></div></td></tr>`;
        return gruppedhtml;
      }
      return trdata;
    },
    fnTabsGroup(tab) {
      var i;
      var x = document.getElementsByClassName("grouptabbtn");
      for (i = 0; i < x.length; i++) {
        x[i].style.display = "none";
      }
      document.getElementById(tab).style.display = "block";
    },
    fnGroupData(groupby) {
      const result = this.datos.reduce(function (r, a) {
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

        if (this.optimized) {
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
      const data = this.fnGroupData(this.groupby);
      const countgroups = Object.keys(data).length;
      if (countgroups < this.limitGroupsQuantity) {
        this.varListGroupsTabsLimitExceded = false;
        this.varListGroupsTabs = Object.keys(data);
      } else {
        this.varListGroupsTabsLimitExceded = true;
      }
    },
  },
  created: function () {
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

th {
  cursor: pointer;
}

/* tabs */
</style>