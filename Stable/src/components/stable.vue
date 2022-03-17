<template>
  <div>
    <select v-model="sortby">
      <option value=""></option>
      <option v-for="(item, index) in headers" :key="index" :value="item.value">
        {{ item.text }}
      </option>
    </select>
    <div>
      <label for="filter">Filtro</label>
      <input type="text" id="filter" v-model="filter" />
    </div>
    <div>
      <input type="button" value="Columnas" />
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
    <table>
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
      <Tbody :html="tbodydata"></Tbody>
      <!-- <tr v-for="(item, index) in tbodydata" :key="index" v-html="item"></tr> -->
    </table>
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
      datos: jsonpoco.data(),
      headers: [
        { value: "_id", text: "ID" },
        { value: "index", text: "indice", type: "number" },
        { value: "guid", text: "guid" },
        { value: "isActive", text: "activo" },
        { value: "balance", text: "balance" },
      ],
      headersSelected: [],
      sortby: null,
      sortasc: false,
      preventOrder: false,
      filter: "",
    };
  },
  computed: {
    tbodydata: function () {
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
      let bodydata = "";
      this.datos.forEach((x) => {
        let item = "";
        this.headersSelected.forEach((element) => {
          item += `<td>${x[element.value]}</td>`;
        });
        if (this.filter) {
          if (item.includes(this.filter)) bodydata += "<tr>" + item + "</tr>";
        } else {
          bodydata += "<tr>" + item + "</tr>";
        }
        // if (this.filter) {
        //   if (item.includes(this.filter)) bodydata.push(item);
        // } else {
        //   bodydata.push(item);
        // }
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
    fnSelectHeaders() {
      // const ref = this.headers.filter((x) => x.value != "actions");
      // this.headers = ref;
      // this.headers.push({
      //   text: "Acciones",
      //   value: "actions",
      //   sortable: false,
      // });
    },
    fnPreventOrder() {
      this.preventOrder = true;
    },
  },
  created: function () {
    this.headersSelected = this.headers;
  },
};
</script>

<style scoped>
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
    -webkit-transform: rotate(90deg);
  }
  to {
    -webkit-transform: rotate(270deg);
  }
}

@keyframes rotationasc {
  from {
    -webkit-transform: rotate(270deg);
  }
  to {
    -webkit-transform: rotate(90deg);
  }
}

th {
  cursor: pointer;
}
</style>