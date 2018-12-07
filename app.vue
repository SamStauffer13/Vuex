
<style>
body {
  text-align: center;
}
</style>

<template>
  <div>
    <input :value="weight" @input="update_weight">
    <input :value="weight_lifted" @input="update_weight_lifted">

    <div>{{ protein }} calories of protein</div>
    <a href="https://vuex.vuejs.org/guide/state.html">using Vuex</a>
  </div>
</template>

<script>
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

// goals:
// model binding of today I lifted
// today I lifted only shows on lift days
// list of random elements is rendered
// add running counter of protein on click of element is incremented

// days till next bulk
// days till next cut
// days till next milestone

const store = new Vuex.Store({
  state: {
    weight: 200,
    weight_lifted: 100
  },
  getters: {
    _calories: (state, getters) => (cut, bulk) => {
      const percentage = getters.body_needs_maitenance ? bulk : cut;
      return Math.round(getters.caloric_intake * percentage);
    },
    body_needs_maitenance: () => {
      const date = new Date();
      const today = date.getDay();
      const maitenanceMonth = (date.getMonth + 1) % 3 === 0;
      const maitenanceDay = 5;
      return today === maitenanceDay || maitenanceMonth;
    },
    caloric_intake: (state, getters) => {
      const percentage = getters.body_needs_maitenance ? 20 : 30;
      return Math.round(state.weight * percentage);
    }
  },
  mutations: {
    _update_weight: (state, weight) => (state.weight = weight),
    _update_weight_lifted: (state, weight) => (state.weight_lifted = weight)
  }
});

export default {
  store,
  components: {},
  computed: {
    ...Vuex.mapState(["weight", "weight_lifted"]),
    ...Vuex.mapGetters(["_calories"]),
    protein() {
      return this._calories(0.25, 0.52);
    },
    carbs() {
      return this._calories(0.3, 0.9);
    },
    fats() {
      return this._calories(0.4, 0.6);
    }
  },
  methods: {
    ...Vuex.mapMutations(["_update_weight", "_update_weight_lifted"]),
    update_weight(e) {
      this._update_weight(e.target.value);
    },
    update_weight_lifted(e) {
      this._update_weight_lifted(e.target.value);
    }
  }
};
</script>
