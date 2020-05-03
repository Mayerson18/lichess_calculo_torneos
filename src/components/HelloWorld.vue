<template>
  <div class="hello">
    <b-field >
      <b-input v-model="input" placeholder="Ingrese el link del torneo"></b-input>
      <p class="control">
        <b-button class="button is-primary" @click="AddTournament">Agregar</b-button>
      </p>
    </b-field>
    <b-button class="button is-primary" @click="sortRanks">Ordenar</b-button>
    <b-table
      :data="data"
      :columns="columns"
    ></b-table>
  </div>
</template>

<script>
  import axios from 'axios';
  export default {
    data() {
      return {
        data: [
          // { 'rank': 1, 'name': 'Jesse', 'score': 0},
          // { 'rank': 2, 'name': 'John', 'score': 0},
          // { 'rank': 3, 'name': 'Tina', 'score': 0},
          // { 'rank': 4, 'name': 'Clarence', 'score': 0},
          // { 'rank': 5, 'name': 'Anne', 'score': 0}
        ],
        columns: [
          {
            field: 'rank',
            label: 'ranking',
            width: '40',
            numeric: true,
            centered: true
          },
          {
            field: 'name',
            label: 'nick',
          },
          {
            field: 'score',
            label: 'Puntos',
            numeric: true
          }
        ]
      }
    },
    methods: {
      sortRanks() {
        this.data.sort(function (a, b) {
          if (a.score < b.score) {
            return 1;
          }
          if (a.score > b.score) {
            return -1;
          }
          // a must be equal to b
          return 0;
        });
      },
      sumTotal(newData) {
        const that = this
        newData.map((dat) => {
          const index = that.data.findIndex(player => player.name === dat.name);
          if (index === -1) {
            that.data.push(dat);
          } else {
            that.data[index].score += dat.score;
          }
        });
      },
      async AddTournament() {
        try {
          
          this.url = this.input;
          const res = await axios.get(this.input);
          let parser = new DOMParser();
          const doc = parser.parseFromString(res.data, "text/html");
          const text = doc.documentElement.getElementsByTagName("script")[2].text;
          const parse = JSON.parse(text.split(";")[1].split("=")[1]);
          const data = parse.data.standing.players;
          console.log('data :>> ', data);
          this.sumTotal(data);

        } catch (error) {
          console.log('error :>> ', error);
          alert("error desconocido wey")
        }
      }
    },
    mounted() {
      // this.data = 
    }
  }
</script>

<style scoped>
.hello {
  padding: 0 5%;
}
</style>

