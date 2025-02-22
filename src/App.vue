<template>
  <header>
    <ProjectLogo/>
    <div class="project-description">
      Rust-Streamerserver von Bonjwa, RocketBeans und Dhalucard
    </div>
    <transition name="slide-y">
      <div v-if="viewers > 1000" class="viewers-container">
        <div class="viewers">
          <span class="heading">total viewers</span>
          <span class="value"><TweenedNumber :value="viewers"/></span>
        </div>
      </div>
    </transition>
  </header>
  <main class="content">
    <section>
      <h1 class="heading">Informationen</h1>
      <ul>
        <li>Der Server ist täglich von 15 Uhr bis 3 Uhr (morgens) online.</li>
        <li>Andere Spieler zu töten ist nur mit Role-Play erlaubt, <b>außer</b> in den sog. KOS-Zonen.</li>
        <li>Um mitspielen zu können, muss ein Streamer von einem der Teilnehmer eingeladen werden.</li>
        <li>Es gibt keinen Global Text-Chat, dafür Team Text-Chat und In-Game Voice-Chat.</li>
        <li>Teams umfassen maximal 6 Personen.</li>
        <li>Der Server wird monatlich zurückgesetzt.</li>
      </ul>
    </section>
    <section>
      <ServerStatus server-is-online/>
    </section>
    <section>
      <h1 class="heading">Teams</h1>
      <div v-if="data === null" class="loading-text">
        Lädt...
      </div>
      <div v-else class="teams">
        <TeamCard v-for="(team, index) in data.teams" :key="index" :team="team"/>
      </div>
      <p>Aktualisiert sich alle 60 Sekunden automatisch.</p>
      <p>Mitspieler erscheinen als online, sobald ihr Kanal live mit Rust ist.</p>
      <p>
        Manche Teams (z. B. Bonjwa) haben nur einen Kanal, aber mehrere Mitspieler. Da es allerdings keine
        einfach Möglichkeit gibt, herauszufinden, wer gerade streamt, erscheinen (fälschlicherweise)
        alle Mitspieler des Teams als live. Nachtrag: Es gibt eine Möglichkeit, aber die würde alles um einiges
        komplizierter machen, weswegen ich es erst mal lasse. Vielleicht habe ich irgendwann ja mal Langeweile...
      </p>
    </section>
  </main>
  <footer class="footer">
    Inoffizielle Seite, erstellt von <a href="https://twitter.com/moritz_ruth">Moritz Ruth</a> |
    <a href="https://github.com/moritzruth/rustplatz">Source Code</a>
  </footer>
</template>

<style>
  .slide-y-enter-active,
  .slide-y-leave-active {
    transition: 1s ease-out;
    transition-property: opacity, max-height;
  }

  .slide-y-enter-to, .slide-y-leave {
    max-height: 145px;
  }

  .slide-y-enter-from,
  .slide-y-leave-to {
    opacity: 0;
    max-height: 0;
  }

  ::selection {
    background: rgba(242, 92, 120, 0.7);
  }

  html {
    font-size: 16px;
  }

  body {
    background: #212121;
    color: #E9E9E9;
    padding: 40px;

    font-family: "Goldman", sans-serif;
    overflow-x: hidden;
  }

  .content {
    max-width: 100%;
    width: 1500px;
    margin: 0 auto;
    font-size: 1.3rem;
  }

  .content p {
    display: block;
    max-width: 800px;
  }

  .content > section {
    margin-bottom: 2em;
  }

  .project-description {
    text-align: center;
    margin-top: 40px;
    font-size: 1.5rem;
  }

  .viewers-container {
    height: 145px;
    width: 400px;
    margin: 20px auto;
  }

  .viewers {
    height: 100%;
    text-align: center;
    max-width: 100%;
    border: 4px dashed #C95847;
    overflow: hidden;
  }

  .viewers > .heading {
    margin-top: 1.4rem;
    font-size: 1.5rem;
    display: block;
    margin-bottom: 0;
  }

  .viewers > .value {
    font-size: 4rem;
    display: block;
    color: #7EFF93;
    margin-bottom: -10px;
  }

  .heading {
    font-weight: bold;
    font-size: 3rem;
    margin-bottom: 20px;
  }

  .loading-text {
    height: 400px;
    font-size: 2rem;
  }

  .teams {
    column-count: 3;
    column-gap: 30px;
  }

  .team-card-container {
    break-inside: avoid;
  }

  @media (max-width: 1500px) {
    .teams {
      column-count: 2;
    }
  }

  @media (max-width: 800px) {
    html {
      font-size: 15px;
    }

    body {
      padding: 40px 20px;
    }

    .teams {
      column-count: 1;
    }

    .heading {
      font-size: 2.4rem;
    }
  }

  ul {
    padding-left: 30px;
  }

  .footer {
    margin-top: 100px;
    width: 100%;
    text-align: center;
  }

  .footer > a {
    display: inline-block;
    text-decoration: none;
    color: #7EFF93;
  }
</style>

<script>
  import ProjectLogo from "./components/ProjectLogo.vue"
  import TeamCard from "./components/TeamCard.vue"
  import TweenedNumber from "./components/TweenedNumber.vue"
  import ServerStatus from "./components/ServerStatus.vue"

  const UPDATE_INTERVAL = 60 * 1000

  export default {
    name: "App",
    components: { ServerStatus, TweenedNumber, TeamCard, ProjectLogo },
    data: () => ({
      data: null
    }),
    computed: {
      viewers: vm => vm.data === null ? 0 : vm.data.totalViewers
    },
    async created() {
      await this.loop()
    },
    methods: {
      async loop() {
        await this.fetchTeams()
        setTimeout(() => {
          this.loop()
        }, UPDATE_INTERVAL)
      },
      async fetchTeams() {
        this.data = process.env.NODE_ENV === "development"
          ? await import("./assets/fake-data").then(m => m.getFakeData())
          : await (await fetch("/.netlify/functions/teams")).json()
      }
    }
  }
</script>
