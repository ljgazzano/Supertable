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
      <Tbody :html="tbodydata" v-if="optimized" style="color: green"></Tbody>
      <tr
        v-else
        v-for="(item, index) in tbodydata"
        :key="index"
        v-html="item"
        style="color: red"
      ></tr>
    </table>

    <div v-else>
      <div class="">
        <button class="" onclick="openCity('London')">London</button>
      </div>

      <div id="London" class="">
        <h2>London</h2>
        <p>London is the capital city of England.</p>
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
      groupby: null,
      headersSelected: [],
      sortby: null,
      sortasc: false,
      preventOrder: false,
      optimized: false,
    };
  },
  computed: {
    tbodydata: function () {
      const countHeaders = this.headersSelected.length;

      if (!this.preventOrder) {
        if (this.sortby) {
          if (this.fnSortByNumber(this.sortby)) {
            this.datos.sort(this.fnOrdenInt);
          } else {
            this.datos.sort(this.fnOrderString);
          }
        }
        if (!this.sortasc) this.datos = this.datos.reverse();
      }

      let bodydata = [];

      this.datos.forEach((x, index) => {
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
    fnTabsGroup(cityName) {
      var i;
      var x = document.getElementsByClassName("city");
      for (i = 0; i < x.length; i++) {
        x[i].style.display = "none";
      }
      document.getElementById(cityName).style.display = "block";
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

/* accordions */
.tdgroupcontainer input {
  display: none;
}

.tdgroupcontainer label {
  display: block;
  padding: 8px 22px;
  margin: 0 0 1px 0;
  cursor: pointer;
  background: #6aab95;
  border-radius: 3px;
  color: #fff;
  transition: ease 0.5s;
}

.tdgroupcontainer label:hover {
  background: #4e8774;
}

.tdgroupcontainer .content {
  background: #e2e5f6;
  padding: 10px 25px;
  border: 1px solid #a7a7a7;
  margin: 0 0 1px 0;
  border-radius: 3px;
}

.tdgroupcontainer input + label + .content {
  display: none;
}

.tdgroupcontainer input:checked + label + .content {
  display: block;
}
</style>