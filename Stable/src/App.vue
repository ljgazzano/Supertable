<script >
import Stable from "./components/stable.vue";
import axios from "axios";

export default {
  components: { Stable },
  data: function () {
    return {
      headers: [
        { value: "category", text: "categoria" },
        { value: "description", text: "descripcion" },
        { value: "id", text: "id", type: "number" },
        { value: "price", text: "precio" },
        { value: "rating", text: "raiting", type: "number" },
        { value: "title", text: "titulo" },
      ],
      items: [],
    };
  },
  methods: {
    async api() {
      const config = {
        url: "https://fakestoreapi.com/products",
        method: "get",
      };
      const datos = await axios(config);
      this.items = datos.data;
    },
  },
};
</script>

<template>
  <div>
    <h2>SuperTable</h2>
    <input type="button" value="Consumir" @click="api()" />
    <Stable
      :headers="headers"
      :items="items"
      :disableOptimized="true"
      :disableFilter="false"
      :disableSort="false"
    />
  </div>
</template>
 