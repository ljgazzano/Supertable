<template>
  <div>
    <select v-model="sortby">
      <option value=""></option>
      <option v-for="(item, index) in headers" :key="index" :value="item.value">
        {{ item.text }}
      </option>
    </select>
    <div>
      <input type="button" value="Columnas" />
    </div>

    <a href="#miModal">Abrir Modal</a>
    <div id="miModal" class="modal">
      <div class="modal-contenido">
        <a href="#">X</a>
        <h2>Selector de Columnas</h2>
        <div>
          <table>
            <tr v-for="(header, index) in headers" :key="index">
              <td>
                <input type="checkbox" @click="headersActions" />{{
                  header.text
                }}
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
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in tbodydata" :key="index" v-html="item"></tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import jsondata from "../assets/jsontest";
import moment from "moment";

export default {
  data: () => {
    return {
      datos: jsondata.data(),
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
    };
  },
  computed: {
    tbodydata: function () {
      if (this.sortby) {
        //   console.log(moment(new Date()).format("YYYY-MM-DDTHH:mm:ss.SSS"));
        if (this.fnSortByNumber(this.sortby)) {
          this.datos.sort(this.fnOrdenInt);
        } else {
          this.datos.sort(this.fnOrderString);
        }
        // console.log(moment(new Date()).format("YYYY-MM-DDTHH:mm:ss.SSS"));
      }

      if (!this.sortasc) {
        this.datos = this.datos.reverse();
      }
      const bodydata = this.datos.map((x) => {
        let item = "";
        this.headers.forEach((element) => {
          item += `<td>${x[element.value]}</td>`;
        });
        return item;
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
      this.sortasc = !this.sortasc;
      this.sortby = val;
    },
  },
  created: function () {
    this.headersSelected = this.headers;
    //  console.log(this.sortby.reverse());
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
</style>