
<style>
body {
  text-align: center;
  font-size: 20px;
}

input {
  outline: none;
  border: 0;
  width: 30px;
  font-size: 15px;
}
</style>

<template>
  <div>
    <div>
      I weigh
      <input :value="weight" @input="update_weight"> pounds
    </div>
    <div>Today I consumed...</div>
    <div @click="update_calories_consumed({protein: 10, fat: 1, carbs: 1})">STEAK</div>

    <div>{{calories_from_protein}} of {{ protein }} calories of protein</div>
    <div>{{calories_from_fat}} of {{ fat }} calories of fats</div>
    <div>{{calories_from_carbs}} of {{ carbs }} calories of carbs</div>
    <br>
    <div v-if="is_workout_day">
      Today I lifted 6 sets of
      <input :value="weight_lifted" @input="update_weight_lifted"> pounds
      <div>next milestone is in {{ weeks_left }} weeks</div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

// goals: list of food elements components

const store = new Vuex.Store({
  state: {
    weight: 150,
    weight_lifted: 100,
    calories_from_protein: 0,
    calories_from_fat: 0,
    calories_from_carbs: 0
  },
  getters: {
    today: () => new Date().getDay(),

    month: () => new Date().getMonth(),

    is_workout_day: (state, getters) => [1, 3, 5].includes(getters.today),

    body_needs_maitenance: (state, getters) =>
      getters.today === 5 || (getters.month + 1) % 3 === 0,

    protein: (state, getters) =>
      getters.algorithm(0.3, 0.9, getters.caloric_intake),

    carbs: (state, getters) =>
      getters.algorithm(0.4, 0.8, getters.caloric_intake),

    fats: (state, getters) =>
      getters.algorithm(0.5, 0.7, getters.caloric_intake),

    caloric_intake: (state, getters) => getters.algorithm(30, 30, state.weight),

    weeks_left: (state, getters) => {
      const goal = 200;

      if (state.weight_lifted >= goal) return 0;

      const whats_left = (goal - state.weight_lifted) / 5;

      return Math.round(whats_left);
    },

    algorithm: (state, getters) => (cut, bulk, amount) => {
      const percentage = getters.body_needs_maitenance ? bulk : cut;

      return Math.round(amount * percentage);
    }
  },
  mutations: {
    _update_weight: (state, weight) => (state.weight = weight),
    _update_weight_lifted: (state, weight) => (state.weight_lifted = weight),

    _add_calories_from_protein: (state, calories) =>
      (state.calories_from_protein += calories),
    _subtract_calories_from_protein: (state, calories) =>
      (state.calories_from_protein -= calories),

    _add_calories_from_fat: (state, calories) =>
      (state.calories_from_fat += calories),
    _subtract_calories_from_fat: (state, calories) =>
      (state.calories_from_fat -= calories),

    _add_calories_from_carbs: (state, calories) =>
      (state.calories_from_carbs += calories),
    _subtract_calories_from_carbs: (state, calories) =>
      (state.calories_from_carbs -= calories)
  }
});

export default {
  store,
  components: {},
  computed: {
    ...Vuex.mapState([
      "weight",
      "weight_lifted",
      "calories_from_protein",
      "calories_from_fat",
      "calories_from_carbs"
    ]),
    ...Vuex.mapGetters([
      "is_workout_day",
      "protein",
      "fats",
      "carbs",
      "weeks_left"
    ])
  },
  methods: {
    ...Vuex.mapMutations([
      "_update_weight",
      "_update_weight_lifted",
      "_add_calories_from_protein",
      "_add_calories_from_fat",
      "_add_calories_from_carbs"
    ]),
    update_weight(e) {
      this._update_weight(e.target.value);
    },
    update_weight_lifted(e) {
      this._update_weight_lifted(e.target.value);
    },
    update_calories_consumed(food) {
      this._add_calories_from_protein(food.protein);
      this._add_calories_from_fat(food.fat);
      this._add_calories_from_carbs(food.carbs);
    }
  }
};
</script>
