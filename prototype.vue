
<style>
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
    </div>    
</template>

<script>
import Vuex from "vuex";
import mapState  from "vuex";
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
//   computed: mapState({
//       count: 'count'
//   }),
  render: function(createElement) {
    return createElement("h2", this.$store.state.count);
  }
};

export default {
  store,
  components: { child1, child2 },
  computed: {
    count: () => store.state.count
  },
  methods: {
    add: () => store.commit("add"),
    remove: () => store.commit("remove")
  }
};
</script>
