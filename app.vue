
<style>
@font-face {
  font-family: "Nikoleta";
  src: url("./NIKOLETA.ttf");
}
body {
  font-family: "Nikoleta";
  text-align: center;
  font-size: 20px;
}

input {
  font-family: "Nikoleta";
  font-weight: bold;
  outline: none;
  border: 0;
  width: 35px;
  font-size: 20px;
}

.food{
   cursor:pointer;
}
</style>

<template>
  <div>
    <div>
      I weigh
      <input :value="weight" @input="update_weight">and want to
      <b @click="update_body_needs_maitenance">{{intent_verbage}}</b> my weight.
    </div>
    <br>
    <div>Today I've consumed {{calories_from_protein + calories_from_fat + calories_from_carbs}} calories</div>
    <span class="food" @click="update_calories_consumed({protein: 233, fat: 367, carbs: 0})">Steak, </span>
    <span class="food" @click="update_calories_consumed({protein: 28, fat: 55, carbs: 29})">Chicken Finger, </span>
    <span class="food" @click="update_calories_consumed({protein: 16, fat: 2, carbs: 142})">Baked Potato, </span>
    <span class="food" @click="update_calories_consumed({protein: 0, fat: 0, carbs: 140})">bottle of coke</span>
    <br>  
    <div v-if="p > 0 || f > 0 || c > 0" >I still need</div>
    <div v-if="p > 0">{{p}} calories of protein</div>
    <div v-if="f > 0">{{f}} calories of fats</div>
    <div v-if="c > 0">{{c}} calories of carbs</div>
    <br>
    <div v-if="p < 0">Overate <span style="color:red"> {{ Math.abs(p) }} </span> calories of protein</div>   
    <div v-if="f < 0">Overate <span style="color:red"> {{ Math.abs(f) }} </span> calories of fat</div>   
    <div v-if="c < 0">Overate <span style="color:red"> {{ Math.abs(c) }} </span> calories of carbs</div>   

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

<<<<<<< HEAD
// goals: list of food elements components
// update 'today I lifted' to include workout excorsize'
// make milestone better: have a timer counting down to the day we reach strength goals!
// make weight better: have a timer counting down to the day we reach weight goals (overeating affects it)!
// keep it up and you'll weigh {target_weight} pounds by {target_date}
// keep it up and you'll be lifting {target_lift_weight} pounds by {target_date}
// scroll to each section (mobile first idea: buttery smooth transitions rather than scrolling)
=======
// goals: list of food elements components + calories turn colors based off numbers + use special font + workout is displayed next to weight goalz
>>>>>>> b5e2efe47f19a8d4c8a13158533c873a3d6f7c34

const store = new Vuex.Store({
  state: {
    weight: 150,
    weight_lifted: 100,
    calories_from_protein: 0,
    calories_from_fat: 0,
    calories_from_carbs: 0,
    is_maitenance_period: false
  },
  getters: {
    today: () => new Date().getDay(),

    month: () => new Date().getMonth(),

    is_workout_day: (state, getters) => [1, 3, 5].includes(getters.today),

    body_needs_maitenance: (state, getters) =>
      state.is_maitenance_period ||
      (getters.today === 5 || (getters.month + 1) % 3 === 0),

    protein: (state, getters) =>
      getters.algorithm(0.27, 0.25, getters.caloric_intake),

    p: (state, getters) =>
      Math.round(getters.protein - state.calories_from_protein),
    c: (state, getters) =>
      Math.round(getters.carbs - state.calories_from_carbs),
    f: (state, getters) =>
      Math.round(getters.fats - state.calories_from_fat),

    carbs: (state, getters) =>
      getters.algorithm(0.45, 0.4, getters.caloric_intake),

    fats: (state, getters) =>
      getters.algorithm(0.33, 0.3, getters.caloric_intake),

    caloric_intake: (state, getters) => getters.algorithm(12, 15, state.weight),

    weeks_left: (state, getters) => {
      const goal = 200;

      if (state.weight_lifted >= goal) return 0;

      const whats_left = (goal - state.weight_lifted) / 5;

      return Math.round(whats_left);
    },

    intent_verbage(state) {
      return state.is_maitenance_period ? "Maintain" : "Decrease";
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
      (state.calories_from_carbs -= calories),

    _update_body_needs_maitenance: state =>
      (state.is_maitenance_period = !state.is_maitenance_period)
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
      "weeks_left",
      "intent_verbage",
      "p", "c", "f"
    ])
  },
  methods: {
    ...Vuex.mapMutations([
      "_update_weight",
      "_update_weight_lifted",
      "_update_body_needs_maitenance",
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
    },
    update_body_needs_maitenance() {
      this._update_body_needs_maitenance();
    }
  }
};
</script>
