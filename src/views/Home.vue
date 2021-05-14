<template>
  <div class="home">
    <div
      v-if="pokes.length > 0"
      style="display: flex;"
    >
      <ul>
        <li
          v-for="poke of sortedPokes"
          :key="poke.id + '_' + Date.now()"
          class="poke-list-item"
          @click="openDetail(poke)"
        >
          <img
            :src="poke.sprites.front_default"
            :alt="poke.name"
            style="width: 80px; height=80px; margin-right: 14px;"
          >

          <div class="poke-details">
            <div class="poke-info">
              <span class="poke-id">#{{poke.id}}</span>
              <span class="poke-name">{{poke.name}}</span>
            </div>

            <div class="types">
              <span
                v-for="typeData of poke.types"
                :key="typeData.type.name"
                :class="`bg--${typeData.type.name}`"
                class="type text--black"
              >
                {{typeData.type.name}}
              </span>
            </div>
            <div
              class="poke-stats"
              v-if="pokeIdToShow === poke.id"
            >
              <ul
                class="stats"
                style="padding: 0;"
              >
                <li
                  v-for="(stat, index) in poke.stats"
                  :key="index"
                >
                  {{ parseStatName(stat.stat.name) }}: {{ stat.base_stat }}
                </li>
              </ul>
              <v-btn
                :loading="btnLoader"
                :disabled="btnLoader"
                @click.prevent="showHowToFindIt(poke.id, $event)"
              >
                Como encontrá-lo
              </v-btn>
              <div>
                <p :ref="`find_${poke.id}`"></p>
              </div>
            </div>
          </div>

        </li>
      </ul>

      <poke-detail
        v-show="showDetail"
        v-bind="stats"
      />
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import PokeDetail from "@/components/PokeDetail.vue"

const statsNames = {
  hp: "HP",
  attack: "Ataque",
  defense: "Defesa",
  speed: "Velocidade",
  "special-attack": "Ataque especial",
  "special-defense": "Defesa especial",
}

export default {
  name: "Home",
  components: {
    PokeDetail,
  },
  mounted() {
    this.fetchUrl(`${this.pokeApiUrl}/pokemon?limit=15&offset=0`)
    // fetch(`${this.pokeApiUrl}/pokemon?limit=15&offset=0`, {
    //   method: "GET",
    //   headers: {
    //     "Content-type": "application/json",
    //   },
    // })
    //   .then((response) => response.json())
    //   .then((result) => {
    //     console.log("result", result)
    //     if (result.results && result.results.length > 0) {
    //       this.getPokes(result.results)
    //       this.nextRound = result.next
    //     }
    //   })
    //   .catch((err) => {
    //     console.log(`err`, err)
    //   })

    setTimeout(() => {
      window.addEventListener("scroll", () => {
        if (
          window.innerHeight + document.documentElement.scrollTop ===
          document.scrollingElement.scrollHeight
        ) {
          this.loadNext()
        }
      })
    }, 3000)
  },
  data() {
    return {
      btnLoader: false,
      pokes: [],
      pokeApiUrl: "https://pokeapi.co/api/v2",
      encounterUrl: "https://pokeapi.co/api/v2/encounter-method/",
      showDetail: false,
      stats: null,
      pokeIdToShow: null,
      nextRound: null,
    }
  },
  computed: {
    sortedPokes() {
      let sorted = this.pokes.sort((a, b) => {
        if (a.id < b.id) {
          return -1
        }
        if (a.id > b.id) {
          return 1
        }

        return 0
      })
      return sorted
    },
  },
  methods: {
    showSpinner(pokeId) {
      this.$refs[`spinner_${pokeId}`][0].style.display = ""
    },
    hideSpinner(pokeId) {
      this.$refs[`spinner_${pokeId}`][0].style.display = "initial"
    },
    showHowToFindIt(pokeId, e) {
      // debugger
      // this.showSpinner(pokeId)
      e.stopPropagation()
      
      this.btnLoader = true
      fetch(this.encounterUrl + pokeId, {
        method: "GET",
        headers: {
          "Content-type": "application/json",
        },
      })
        .then((response) => response.json())
        .then((result) => {
          console.log("result", result)
          if (result.names && result.names.length > 0) {
            const filterEn = result.names.filter((method) => {
              if (method.language.name == "en") return method
            })
            if (filterEn && filterEn.length > 0) {
              this.$refs[`find_${pokeId}`][0].innerText = filterEn[0].name
            }
            setTimeout(() => {
              this.btnLoader = false
              if (filterEn && filterEn.length > 0) {
                this.$nextTick(() => {
                  this.$refs[`find_${pokeId}`][0].innerText = filterEn[0].name
                })
              }
            }, 250)
          }
        })
        .catch((err) => {
          console.log(`err`, err)
        })
    },
    fetchUrl(url) {
      fetch(url, {
        method: "GET",
        headers: {
          "Content-type": "application/json",
        },
      })
        .then((response) => response.json())
        .then((result) => {
          console.log("result", result)
          if (result.results && result.results.length > 0) {
            this.getPokes(result.results)
            this.nextRound = result.next
          }
        })
        .catch((err) => {
          console.log(`err`, err)
        })
    },
    loadNext() {
      if (this.nextRound) {
        this.fetchUrl(this.nextRound)
      }
    },
    parseStatName(name) {
      return statsNames[name] || name
    },
    openDetail(poke) {
      // this.showDetail = true
      // this.stats = poke.stats
      if (this.pokeIdToShow != poke.id) this.pokeIdToShow = poke.id
      else this.pokeIdToShow = null
    },
    getPokes(list) {
      if (list && list.length > 0) {
        for (let poke of list) {
          fetch(`${this.pokeApiUrl}/pokemon/${poke.name}`, {
            method: "GET",
            headers: {
              "Content-type": "application/json",
            },
          })
            .then((response) => response.json())
            .then((result) => {
              console.log("poke by name", result)
              this.pokes.push(result)
            })
            .catch((err) => {
              console.log(`err`, err)
            })
        }
      }
    },
  },
}
</script>

<style>
.stats {
  list-style-type: none;
}
.poke-details {
  display: flex;
  flex-direction: column;
  flex: 1;
  font-size: 14px;
  padding: 0 8px;
}
.poke-name {
  text-transform: capitalize;
}
.poke-list-item {
  list-style-type: none;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 4px;
  cursor: pointer;
  margin: 2% 0;
}
.poke-info {
  display: flex;
  flex-direction: column;
}
.types {
  display: flex;
  font-size: 12px;
  justify-content: space-evenly;
}
.type {
  padding: 0 8px;
  font-weight: bold;
}
</style>

<style>
.bg--bug {
  background-color: #009c19;
}
.bg--dark {
  background-color: #707070;
}
.bg--dragon {
  background-color: #4693bd;
}
.bg--electric {
  background-color: #eed535;
}
.bg--fairy {
  background-color: #fdb9e9;
}
.bg--fighting {
  background-color: #c75c1a;
}
.bg--fire {
  background-color: #fd5724;
}
.bg--flying {
  background-color: #52d2f5;
}
.bg--ghost {
  background-color: #7b62a3;
}
.bg--grass {
  background-color: #9bcc50;
}
.bg--ground {
  background-color: #c5b759;
}
.bg--ice {
  background-color: #5ad2f6;
}
.bg--normal {
  background-color: #a4acaf;
}
.bg--poison {
  background-color: #b97fc9;
}
.bg--psychic {
  background-color: #f366b9;
}
.bg--rock {
  background-color: #807940;
}
.bg--steel {
  background-color: #a5b2b3;
}
.bg--water {
  background-color: #2780d4;
}
</style>
