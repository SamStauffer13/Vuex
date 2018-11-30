
<style>
body {
  text-align: center;
}
ol {
  max-width: 25em;
  text-align: left;
  margin: 0 auto;
}
</style>

<template>
    <div>       
    <child1></child1>
    <child2></child2>
    <h3> {{ count }}</h3>
    <div @click="add"> Add </div>
    <div @click="remove"> Remove </div>
    <br>
    <a href="https://vuex.vuejs.org/guide/state.html"> using Vuex </a>
    <br>
    <h1>Goalz</h1>
    <ol>
      <li>Top off water in Aquarium</li>
      <li>Load dishwasher</li>
      <li>Mount sidebar to undercarrage of desk</li>      
      <li>Hardware store: drill press and furnace filter</li>
      <li>Mental and Physical health app, background lofi video div with transparent todo app overlay.</li>
    </ol>
    </div>    
</template>

<script>
import Vuex from "vuex";
import Vue from "vue";

Vue.use(Vuex);

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    add: state => state.count++,
    remove: state => state.count--
  }
});

const child1 = {
  render: function(createElement) {
    return createElement("h1", this.$store.state.count);
  }
};

const child2 = {
  computed: Vuex.mapState(["count"]),
  render: function(createElement) {
    return createElement("h2", this.count);
  }
};

export default {
  store,
  components: { child1, child2 },
  computed: Vuex.mapState(["count"]),
  methods: Vuex.mapMutations(["add", "remove"])
};

// todo
</script>
