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
        id: '',
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
          this.id = this.input.split("/")[4];
          this.url = this.input;
          const res = await axios.get(this.input);
          /* DATA PRINCIPAL DEL TORNEO */
          let parser = new DOMParser();
          const doc = parser.parseFromString(res.data, "text/html");
          const text = doc.documentElement.getElementsByTagName("script")[2].text;
          const parse = JSON.parse(text.split(";")[1].split("=")[1]);
          const players = parse.data.nbPlayers;
          const pages = Math.ceil(players/10);
          const data = parse.data.standing.players;
          if (pages > 1) {
            
            for (let i = 2; i <= pages; i++) {
              const res2 = await axios.get(this.input + "/standing/" + i);
              this.sumTotal(res2.data.players);
            }
          }
          this.sumTotal(data);
        } catch (error) {
          console.log('error :>> ', error);
          alert("error desconocido wey")
        }
      }
    },
    async mounted() {
      // this.data = 
      // const res = await axios.get("https://lichess.org/tournament/rO2jbTK0/standing/2?_=1588484557270");
      // console.log('res :>> ', res);
    }
  }
</script>

<style scoped>
.hello {
  padding: 0 5%;
}
</style>

