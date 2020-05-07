<template>
  <div class="hello">
    <div class="flex-between">
      <b-field >
        <b-input v-model="input" placeholder="Ingrese el link del torneo"></b-input>
        <p class="control">
          <b-button class="button is-primary" @click="AddTournament">Agregar</b-button>
        </p>
      </b-field>
      <!-- <b-field >
        <b-input v-model="input" placeholder="Ingrese el link del equipo"></b-input>
        <p class="control">
          <b-button class="button is-primary" @click="addTeam">Agregar</b-button>
        </p>
      </b-field> -->
    </div>
    <b-collapse
      aria-id="contentIdForA11y2"
      class="panel"
      animation="slide"
      :open.sync="isOpen">
      <div
        slot="trigger"
        class="panel-heading"
        role="button"
        aria-controls="contentIdForA11y2">
        <strong>Torneos</strong>
      </div>
      <div class="panel-block flex-center">
        <div v-for="(tournament, index) of tournaments" :key="index" class="flex-between w60">
          <button class="button is-primary"
            @click="detailTournament(tournament)"
          >
            {{tournament.fullName}}
          </button>
          <button class="button is-primary"
            @click="sumTotal(tournament)">
              Sumar Torneo
          </button>
          <button class="button is-danger"
            @click="removeTournament(tournament.id)">
              Eliminar Torneo
          </button>
        </div>
      </div>
    </b-collapse>
    <b-collapse
      aria-id="contentIdForA11y2"
      class="panel"
      animation="slide"
      :open.sync="isOpen2">
      <div
        slot="trigger"
        class="panel-heading"
        role="button"
        aria-controls="contentIdForA11y2">
        <strong>Resultados</strong>
      </div>
      <div class="panel-block flex-center">
        <div v-for="(result, index) of results" :key="index" class="flex-between w60">
          <button class="button is-primary"
            @click="showResult(result)"
          >
            {{result.fullName}}
          </button>
          <button class="button is-danger"
            @click="removeResult(result.id)">
              Eliminar Resultado
          </button>
        </div>
      </div>
    </b-collapse>
    <div class="flex-between">
      <b-button class="button is-primary" @click="sortRanks">Ordenar</b-button>
      <b-button class="button is-success" @click="openResult">Guardar Resultado</b-button>
      <!-- <b-button class="button is-primary" @click="sortRanks">Ordenar</b-button> -->
    </div>
    <b-table
      :data="data"
      :columns="columns"
    ></b-table>
    <sweet-modal ref="modalResult">This is an alert</sweet-modal>
  </div>
</template>

<script>
  import { SweetModal } from 'sweet-modal-vue'
  import axios from 'axios';
  export default {
    components: {
      SweetModal
    },
    data() {
      return {
        isOpen: true,
        isOpen2: true,
        inputTeams: '',
        input: '',
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
            field: 'tournaments',
            label: 'Torneos jugados',
          },
          {
            field: 'score',
            label: 'Puntos',
            numeric: true
          }
        ],
        tournaments: [],
        tournament_selected: {},
        teams: [],
        results: []
      }
    },
    methods: {
      saveResult() {
        this.results = [...this.results, this.data];
        localStorage.setItem('results', JSON.stringify(this.data))
      },
      openResult() {
        this.$refs.modalResult.open();
      },
      addTeam() {
        console.log('object :>> ');
      },
      removeTournament(id) {
        let tournaments = JSON.parse(localStorage.getItem('tournaments'));
        tournaments = tournaments.filter((t) => t.id !== id);
        localStorage.setItem('tournaments', JSON.stringify(tournaments));
        this.tournaments = tournaments;
      },
      detailTournament (tournament) {
        this.tournament_selected = tournament;
        console.log('tournament :>> ', tournament);
      },
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
      sumTotalTournament(data) {
        const that = this
        data.map((dat) => {
          const index = that.data.findIndex(player => player.name === dat.name);
          if (index === -1) {
            dat.tournaments = 0;
            that.data.push(dat);
          } else {
            that.data[index].score += dat.score;
            that.data[index].tournaments++;
          }
        });
      },
      async sumTotal(tournament) {
        console.log('tournament :>> ', tournament);
        const players = tournament.nbPlayers;
        const pages = Math.ceil(players/10);
        const data = tournament.standing.players;
        if (pages > 1) {
          for (let i = 2; i <= pages; i++) {
            const res2 = await axios.get("https://lichess.org/tournament/" + tournament.id + "/standing/" + i);
            this.sumTotalTournament(res2.data.players);
          }
        }
        this.sumTotal(data);
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
          const tournament = JSON.parse(text.split(";")[1].split("=")[1]);
          console.log('parse :>> ', tournament);
          this.tournaments.push(tournament.data);
          localStorage.setItem('tournaments', JSON.stringify(this.tournaments))
        } catch (error) {
          console.log('error :>> ', error);
          alert("error desconocido wey")
        }
      }
    },
    async mounted() {
      let tournaments = localStorage.getItem('tournaments');
      if (tournaments) {
        this.tournaments = JSON.parse(tournaments);
      }
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

.flex-between {
  display: flex;
  justify-content: space-between;
  margin: 2em 0;
}

.w60 {
  width: 60%;
}

.flex-center {
  display: flex;
  justify-content: center;
}
</style>

