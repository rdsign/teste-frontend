<template>
  <main class="Mission">
    <div v-if="!readyComp" class="loader">
      <Logo />
    </div>

    <template v-else>
      <MainHeader />
      <BackgroundGraphic />

      <section class="intro container">
        <div class="thumb">
          <img class="lens" src="~/assets/images/thumb_lens.png" />
          <img v-if="mission.thumb_url" :src="mission.thumb_url" :alt="mission.name" />
          <img v-else src="~/assets/images/thumb_mission.jpg" :alt="mission.name" />
        </div>
        <div class="summary">
          <ul>
            <li>
              <NLink to="/">skore</NLink>
            </li>
            <span>/</span>
            <li>
              <NLink to="/missions">missões</NLink>
            </li>
          </ul>
          <h1>{{ mission.name }}</h1>
          <h4>Nome da Empresa</h4>
          <p v-if="mission.due_date">
            Encerra em:
            {{
            new Date(
            mission.due_date.available_at
            ).toLocaleDateString("en-US", { dateStyle: "short" })
            }}
          </p>
        </div>
      </section>

      <section class="step-list container">
        <SectionListFilter title="Suas missões" :active="''" :filters="[]" />
        <transition-group tag="ul" class="list" name="cardAnimation">
          <Progress
            v-for="(step, indx) in mission.steps"
            :key="step.id"
            :style="`order: ${indx}`"
            :name="step.name"
            :progress="Math.random() * 100"
          />
        </transition-group>
      </section>
    </template>
  </main>
</template>

<script>
import { mapState } from "vuex";
import BackgroundGraphic from "~/components/UI/BackgroundGraphic";
import Logo from "~/components/Logo";
import MainHeader from "~/components/MainHeader/MainHeader";
import Progress from "~/components/UI/Progress";
import SectionListFilter from "~/components/SectionListFilter/SectionListFilter";

export default {
  components: {
    BackgroundGraphic,
    Logo,
    MainHeader,
    Progress,
    SectionListFilter
  },

  head() {
    return {
      title: "Missão",
      meta: [
        {
          hid: "description",
          name: "description",
          content: "Mission Name"
        }
      ]
    };
  },

  data() {
    return {
      ready: false,
      erro: false
    };
  },

  computed: {
    ...mapState({ missions: "missions", mission: "selectedMission" }),
    readyComp() {
      return this.ready;
    },
    erroComp() {
      return this.erro;
    }
  },

  methods: {
    getMissions() {
      this.$axios
        .$get(
          "https://us-central1-teste-frontend-c2dcc.cloudfunctions.net/missions"
        )
        .then(
          res => (
            this.$store.dispatch("loadMissions", {
              data: res,
              context: this
            }),
            this.checker()
          )
        );
    },
    getMission() {
      this.$axios
        .$get(
          `https://us-central1-teste-frontend-c2dcc.cloudfunctions.net/missions/${this.$route.params.mission}`
        )
        .then(
          res => (
            this.$store.dispatch("loadMission", {
              data: res,
              context: this
            }),
            (this.ready = true)
          )
        );
    },
    checker() {
      const validate = this.$store.state.missions.some(
        m => m.id === this.$route.params.mission
      );

      if (validate) {
        this.$axios
          .$get(
            `https://us-central1-teste-frontend-c2dcc.cloudfunctions.net/missions/${this.$route.params.mission}`
          )
          .then(
            res => (
              this.$store.dispatch("loadMission", {
                data: res,
                context: this
              }),
              (this.ready = true)
            )
          );
      } else {
        this.$router.push({
          path: "/error"
        });
      }
    }
  },

  async created() {
    if (this.$store.state.missions.length == 0) {
      await this.getMissions();
    } else {
      await this.getMission();
    }
  }
};
</script>

<style lang="scss">
@import "@/assets/style/_colors.scss";
@import "@/assets/style/_variables.scss";

.Mission {
  & .loader {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);

    & .Logo {
      width: 20vw;
      animation: loader 2s linear;
      animation-iteration-count: infinite;
      animation-delay: 1s;
      opacity: 0;
    }
  }

  & .intro {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 3vh;

    & .thumb {
      position: relative;
      height: 35vh;
      flex: 20;
      border-radius: 6px;
      box-shadow: 0px 1vw 3vw rgba(black, 0.14);
      animation: 0.8s down-top ease-out;
      animation-iteration-count: 1;

      & img {
        object-fit: cover;
        width: 100%;
        height: 100%;
        border-radius: 6px;
      }

      & .lens {
        position: absolute;
        top: 0;
        left: 0;
      }
    }

    & .summary {
      flex: 80;
      margin-top: -6vh;
      margin-left: 3vw;

      & ul {
        display: flex;
        align-items: center;
        padding: 0 0 2.5vh;
        animation: 0.8s appear ease-out;
        animation-iteration-count: 1;

        & * {
          color: $color_primary;
          font-weight: bold;
          letter-spacing: 0.1em;
        }

        & li {
          padding: 0 0.6vw;

          & a {
            text-transform: uppercase;
            font-size: $p;

            &:hover {
              color: $color_light;
              background-color: $color_primary;
            }
          }
        }

        & li:nth-child(1) {
          padding-left: 0;
        }
      }
      & h1 {
        font-size: calc(#{$p} * 3);
        letter-spacing: -0.05em;
        animation: 0.8s appear ease-out;
        animation-iteration-count: 1;
      }
      & h4 {
        margin-top: 0.5vh;
        font-size: calc(#{$p} * 1.8);
        font-weight: 200;
        letter-spacing: -0.05em;
        animation: 0.8s appear ease-out;
        animation-iteration-count: 1;
      }
      & p {
        margin-top: 2.5vh;
        font-size: calc(#{$p} * 1.2);
        font-weight: 200;
        letter-spacing: -0.05em;
        animation: 0.8s appear ease-out;
        animation-iteration-count: 1;
      }
    }
  }
  & .step-list {
    margin-top: 15vh;
    animation: 0.8s appear ease-out;
    animation-iteration-count: 1;
    min-height: 50vh;

    & .filter {
      display: none;
    }

    & .list {
      display: flex;
      flex-wrap: wrap;
      margin-top: 8vh;

      & .Progress {
        width: 100%;
        margin-bottom: 2%;
      }
    }
  }
}

.cardAnimation-enter-active {
  transition: all 0.3s ease;
}
.cardAnimation-leave-active {
  transition: all 0.6s cubic-bezier(1, 0.5, 0.8, 1);
}
.cardAnimation-enter,
.cardAnimation-leave-to {
  transform: translateY(10px);
  opacity: 0;
}

@keyframes down-top {
  0% {
    margin-top: 15vh;
    opacity: 0;
  }
  100% {
    margin-top: 0;
    opacity: 1;
  }
}

@keyframes loader {
  0% {
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 0;
    transform: scale(1.2);
  }
}

@media all and (min-width: 280px) and (max-width: 960px) and (orientation: portrait) {
  .Mission {
    & .intro {
      justify-content: center;
      align-items: flex-start;

      & .thumb {
        min-width: 20vw;
        max-width: 20vw;
        height: 20vh;
        flex: unset;
      }

      & .summary {
        flex: unset;
        margin-top: 0;
        margin-left: 5vw;

        & ul {
          padding: 0 0 1.5vh;

          & li {
            padding: 0 2vw;
          }
        }
        & h1 {
          font-size: calc(#{$p} * 1.8);
        }
        & h4 {
          font-size: calc(#{$p} * 1.2);
        }
        & p {
          margin-top: 1.5vh;
          font-size: calc(#{$p} * 0.8);
        }
      }
    }
    & .step-list {
      margin-top: 15vh;
      min-height: 50vh;

      & .list {
        margin-top: 6vh;
      }
    }
  }
}
@media all and (min-height: 280px) and (max-height: 640px) and (orientation: landscape) {
  .Mission {
    & .intro {
      justify-content: center;

      & .thumb {
        min-width: 15vw;
        max-width: 15vw;
        height: 40vh;
        flex: unset;
      }

      & .summary {
        flex: unset;
        margin-top: 0;
        margin-left: 5vw;

        & ul {
          padding: 0 0 1.5vh;

          & li {
            padding: 0 2vw;
          }
        }
        & h1 {
          font-size: calc(#{$p} * 2.1);
        }
        & h4 {
          font-size: calc(#{$p} * 1.5);
        }
        & p {
          margin-top: 1.5vh;
          font-size: calc(#{$p} * 1);
        }
      }
    }
    & .step-list {
      margin-top: 30vh;
      min-height: 80vh;

      & .list {
        max-width: 80%;
        margin: 0 auto;
        margin-top: 8vh;
      }
    }
  }
}
</style>
