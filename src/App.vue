<template>
  <div id="app">
    <nav class="bar static transparent">
      <div class="container">
        <ul>
          <li class="collapse"><a href="#"><i class="fa fa-bars"></i></a></li>
          <li class="brand">
            <a href="https://diamondpkg.org">
              <img src="./assets/Diamond.svg" width="30px" height="26px">
            </a>
          </li>
          <ul class="right">
            <li>
              <a href="https://github.com/diamondpkg/diamond" target="_blank">
                <i class="fa fa-github fa-fw"></i>
              </a>
            </li>
          </ul>
        </ul>
      </div>
    </nav>

    <div :class="['header', 'hero', { 'full-height': !maintenance.length && !current.concat(incidents).length }]" :style="{ background: color }">
      <div class="container">
        <img class="logo" src="./assets/Diamond.svg" height="128px" width="128px">
        <h1>{{ status }}</h1>
        <h2 v-if="maintenance.length">Maintenance {{ moment(parseInt(maintenance[0].title.split('!')[1], 10)).fromNow() }}</h2>
        <h5 v-if="responseTime">Registry responded in {{ responseTime }}ms</h5>
      </div>
    </div>

    <main>
      <div class="row">
        <div v-if="maintenance.length" class="box col-6">
          <h1>Upcoming Maintenance</h1>
          <div class="card" v-for="incident of maintenance">
            <div class="content">
              <b>{{ incident.title.split('!')[0] }}</b> &nbsp;
              <span class="tag square" :style="{ background: `#${incident.labels[0].color}`, color: 'white' }">{{ incident.labels[0].name }}</span> &nbsp;
              <span class="right">{{ moment(parseInt(incident.title.split('!')[1], 10)).format('MMMM Do YYYY, h:mm:ss a') }}</span>

              <p>{{ incident.body }}</p>
              <div class="links" align="right">
                <a :href="incident.html_url">View on GitHub</a>
              </div>
            </div>
          </div>
        </div>

        <div v-if="current.concat(incidents).length" class="box col-6">
          <h1>Incidents</h1>
          <div class="card" v-for="incident of current.concat(incidents)">
            <div class="content">
              <b>{{ incident.title.split('!')[0] }}</b> &nbsp;
              <span class="tag square" :style="{ background: `#${incident.labels[0].color}`, color: 'white' }">{{ incident.labels[0].name }}</span> &nbsp;
              <span class="right">{{ moment(incident.created_at).format('MMMM Do YYYY, h:mm:ss a') }}</span>

              <p>{{ incident.body }}</p>
              <div class="links" align="right">
                <a :href="incident.html_url">View on GitHub</a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import moment from 'moment';
import superagent from 'superagent';

export default {
  name: 'app',
  data() {
    return {
      moment,
      incidents: [],
      current: [],
      maintenance: [],
      responseTime: null,
      status: 'Getting Status...',
      color: '#3F3F3F',
    };
  },
  async mounted() {
    const req = await superagent.get('https://api.github.com/repos/diamondpkg/status/issues?state=all&creator=Hackzzila');

    this.$data.incidents = req.body.filter(x => x.state === 'closed');
    this.$data.current = req.body.filter((x) => {
      if (x.state === 'closed') return false;
      if (x.labels[0].name === 'Maintenance' && parseInt(x.title.split('!')[1], 10) < Date.now()) return true;
      if (x.labels[0].name === 'Maintenance') return false;
      return true;
    });
    this.$data.maintenance = req.body.filter(x => x.state !== 'closed' && x.labels[0].name === 'Maintenance' && parseInt(x.title.split('!')[1], 10) > Date.now()).sort((a, b) => parseInt(a.title.split('!')[1], 10) > parseInt(b.title.split('!')[1], 10));

    if (this.$data.current.length === 1) {
      this.$data.color = `#${this.$data.current[0].labels[0].color}`;
      this.$data.status = this.$data.current[0].labels[0].name;
    } else if (this.$data.current.length) {
      this.$data.color = '#FF4D4D';
      this.$data.status = 'Multiple Issues';
    } else {
      this.$data.color = '#52C652';
      this.$data.status = 'All Systems Operational';
    }
  },
};
</script>

<style lang="sass">
  @import '~caramel'

  .header.hero
    color: $white
    transition: background 200ms ease-in-out
</style>
