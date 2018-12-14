
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
  outline: none;
  border: 0;
}

.weight {
  font-weight: bold;
  width: 35px;
  font-size: 20px;
}

.search {
  width: 200px;
  font-size: 20px;
}

.failed {
  color: red;
}

.clickable {
  cursor: pointer;
}
</style>

<template>
  <div>
    <div>
      I weigh
      <input class="weight" :value="weight" @input="_updateWeight">and want to
      <b>decrease</b> my weight.
    </div>
    <br>
    <div>
      which means I need to consume about
      <b>{{caloricIntake}}</b> calories today
    </div>
    <br>

    <div>
      Today I consumed
      <span :class="{failed: caloriesConsumed > caloricIntake}">{{caloriesConsumed}}</span> calories from...
    </div>

    <div v-for="food in foodzConsumed" :key="food.name+food.qty">
      <span>{{food.qty}} {{food.name}}</span> (
      <span class="clickable" @click="addFoodzEaten(food)">+</span>/
      <span class="clickable" @click="subtractFoodzEaten(food)">-</span>)
    </div>
    <br>

    <input
      class="search"
      :value="search"
      @input="_updateSearch"
      placeholder="Search Foodz..."
      autofocus
    >

    <div v-for="food in foodz" :key="food.name">
      <span class="clickable" @click="addFoodzEaten(food)">{{food.name}}</span>
    </div>
    <br>
    <br>

    <div>I still need
      <div v-if="proteinRemaining > 0">{{proteinRemaining}} calories of protein</div>
      <div v-if="carbsRemaining > 0">{{carbsRemaining}} calories of carbs</div>
      <div v-if="fatRemaining > 0">{{fatRemaining}} calories of fat</div>
      <br>
      <div v-if="proteinRemaining < 0">
        Overate protein by
        <span style="color:red;">{{ Math.abs(proteinRemaining)}}</span> calories
      </div>
      <div v-if="carbsRemaining < 0">
        Overate carbs by
        <span style="color:red;">{{ Math.abs(carbsRemaining)}}</span> calories
      </div>
      <div v-if="fatRemaining < 0">
        Overate fat by
        <span style="color:red;">{{ Math.abs(fatRemaining)}}</span>
        calories
      </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import Vuex from "vuex";

// todoz
// impliment constants instead of magic strings?
// take a closer look at model binding, I think it's modifying state when it shouldn't
// use es6 reducer instead of foreach loops
// show caloric deficit amount explanation next to calories per day
// add a 'hey you need to burn x number of calories to get back to where you need to be, modified by today I lifed and today I ran
// add a food to the foodz list function
// add a strikethrough to protein/fat/carb if we've exceeded daily calories

Vue.use(Vuex);

class Food {
  constructor(name, protein, fats, carbs, qty = 0) {
    this.name = name;
    this.protein = protein;
    this.carbs = carbs;
    this.fats = fats;
    this.qty = qty;
    this.aliases = []; // for fuzzy search
  }
}

const f = [
  new Food("Steak", 233, 367, 0),
  new Food("Chicken Finger", 28, 55, 29),
  new Food("Baked Potato", 16, 2, 142),
  new Food("Glass Of Coke", 0, 0, 142),

  new Food("Slice of Pizza", 0, 0, 0),
  new Food("Glass of Bourbon", 0, 0, 0),
  new Food("Handful of Chips", 0, 0, 0)
];

const store = new Vuex.Store({
  state: {
    weight: 150,
    foods: f,
    search: ""
  },
  getters: {
    today: () => new Date().getDay(),

    month: () => new Date().getMonth(),

    needsWorkout: (state, getters) => [1, 3, 5].includes(getters.today),

    isRefillDay(state, getters) {
      return getters.today === 5;
    },
    isRefillMonth(state, getters) {
      return (getters.month + 1) % 3 === 0;
    },
    needsCalories(state, getters) {
      return getters.isRefillDay || getters.isRefillMonth;
    },
    protein(state, getters) {
      return getters.algorithm(0.27, 0.25, getters.caloricIntake);
    },
    proteinConsumed(state, getters) {
      let sum = 0;
      getters.foodzConsumed.forEach(food => {
        sum += food.protein * food.qty;
      });
      return sum;
    },
    proteinRemaining(state, getters) {
      return Math.round(getters.protein - getters.proteinConsumed);
    },
    carbs(state, getters) {
      return getters.algorithm(0.45, 0.4, getters.caloricIntake);
    },
    carbsConsumed(state, getters) {
      let sum = 0;
      getters.foodzConsumed.forEach(food => {
        sum += food.carbs * food.qty;
      });
      return sum;
    },
    carbsRemaining(state, getters) {
      return Math.round(getters.carbs - getters.carbsConsumed);
    },
    fats(state, getters) {
      return getters.algorithm(0.33, 0.3, getters.caloricIntake);
    },
    fatsConsumed(state, getters) {
      let sum = 0;
      getters.foodzConsumed.forEach(food => {
        sum += food.fats * food.qty;
      });
      return sum;
    },
    fatRemaining(state, getters) {
      return Math.round(getters.fats - getters.fatsConsumed);
    },
    caloriesConsumed(state, getters) {
      return (
        getters.proteinConsumed + getters.carbsConsumed + getters.fatsConsumed
      );
    },
    caloriesRemaining(state, getters) {
      return getters.caloricIntake - getters.caloriesConsumed;
    },
    caloricIntake(state, getters) {
      return getters.algorithm(12, 15, state.weight);
    },
    algorithm: (state, getters) => (a, b, c) => {
      return Math.round(c * (getters.needsCalories ? a : b));
    },
    foodz(state) {
      if (state.search === "") return [];
      return state.foods.filter(food =>
        food.name.toLowerCase().includes(state.search.toLowerCase())
      );
    },
    foodzConsumed(state) {
      return state.foods.filter(food => food.qty > 0);
    }
  },
  mutations: {
    updateWeight(state, weight) {
      state.weight = weight;
    },
    updateSearch(state, search) {
      state.search = search;
    },
    addFoodzEaten(state, food) {
      state.foods.find(meal => meal.name === food.name).qty++;
    },
    subtractFoodzEaten(state, food) {
      state.foods.find(meal => meal.name === food.name).qty--;
    }
  }
});

export default {
  store,
  components: {},
  computed: {
    ...Vuex.mapState(["weight", "search"]),
    ...Vuex.mapGetters([
      "caloricIntake",
      "fatRemaining",
      "proteinRemaining",
      "carbsRemaining",
      "caloriesConsumed",
      "caloriesRemaining",
      "foodz",
      "foodzConsumed"
    ])
  },
  methods: {
    ...Vuex.mapMutations([
      "updateWeight",
      "updateSearch",
      "addFoodzEaten",
      "subtractFoodzEaten"
    ]),
    _updateWeight(e) {
      this.updateWeight(e.target.value);
    },
    _updateSearch(e) {
      this.updateSearch(e.target.value);
    }
  }
};
</script>
