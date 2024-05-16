<template>
  <main v-if="dataStore.isDataLoaded" class="content">
    <form action="#" method="post">
      <div class="content__wrapper">
        <h1 class="title title--big">Конструктор пиццы</h1>

        <dough-selector v-model="doughId" :items="dataStore.doughs" />

        <diameter-selector v-model="sizeId" :items="dataStore.sizes" />

        <div class="content__ingredients">
          <div class="sheet">
            <h2 class="title title--small sheet__title">
              Выберите ингредиенты
            </h2>

            <div class="sheet__content ingredients">
              <sauce-selector v-model="sauceId" :items="dataStore.sauces" />

              <ingredients-selector :values="pizzaStore.ingredientQuantities" :items="dataStore.ingredients"
                @update="pizzaStore.setIngredientQuantity" />

            </div>
          </div>
        </div>

        <div class="content__pizza">
          <label class="input">
            <span class="visually-hidden">Название пиццы</span>
            <input v-model="name" type="text" name="pizza_name" placeholder="Введите название пиццы" />
          </label>

          <pizza-constructor :dough="pizzaStore.dough.value" :sauce="pizzaStore.sauce.value"
            :ingredients="pizzaStore.ingredientsExtended" @drop="pizzaStore.incrementIngredientQuantity" />

          <div class="content__result">
            <p>Итого: 0 ₽</p>
            <button type="button" class="button" :disabled="disableSubmit" @click="addToCart">Готовьте!</button>
          </div>
        </div>
      </div>
    </form>
  </main>
</template>

<script setup>
import { computed, onMounted, reactive } from "vue";
import { useRouter } from "vue-router";

import DoughSelector from "@/modules/constructor/DoughSelector.vue";
import DiameterSelector from "@/modules/constructor/DiameterSelector.vue";
import SauceSelector from "@/modules/constructor/SauceSelector.vue";
import IngredientsSelector from "@/modules/constructor/IngredientsSelector.vue";
import PizzaConstructor from "@/modules/constructor/PizzaConstructor.vue";

import doughJSON from "@/mocks/dough.json";
import ingredientsJSON from "@/mocks/ingredients.json";
import saucesJSON from "@/mocks/sauces.json";
import sizesJSON from "@/mocks/sizes.json";
import {
  normalizeDough,
  normalizeIngredients,
  normalizeSauces,
  normalizeSize,
} from "@/common/helpers/normalize";

import { useDataStore } from "@/stores/data";
import { usePizzaStore } from "@/stores/pizza";
import { useCartStore } from "@/stores/cart";

const dataStore = useDataStore();
const pizzaStore = usePizzaStore();
const cartStore = useCartStore();

const router = useRouter()

const doughItems = doughJSON.map(normalizeDough);
const ingredientItems = ingredientsJSON.map(normalizeIngredients);
const sauceItems = saucesJSON.map(normalizeSauces);
const sizeItems = sizesJSON.map(normalizeSize);

onMounted(() => {
  if (pizzaStore.index === null) {
    resetPizza();
  }
});

const name = computed({
  get() {
    return pizzaStore.name;
  },
  set(value) {
    pizzaStore.setName(value);
  },
});

const doughId = computed({
  get() {
    return pizzaStore.doughId;
  },
  set(value) {
    pizzaStore.setDough(value);
  },
});

const sizeId = computed({
  get() {
    return pizzaStore.sizeId;
  },
  set(value) {
    pizzaStore.setSize(value);
  },
});

const sauceId = computed({
  get() {
    return pizzaStore.sauceId;
  },
  set(value) {
    pizzaStore.setSauce(value);
  },
});

const pizza = reactive({
  name: "",
  dough: doughItems[0].value,
  size: sizeItems[0].value,
  sauce: sauceItems[0].value,
  ingredients: ingredientItems.reduce((acc, item) => {
    acc[item.value] = 0;
    return acc;
  }, {}),
});

const disableSubmit = computed(() => {
  return name.value.length === 0 || pizzaStore.price === 0;
});

const addToCart = async () => {
  cartStore.savePizza(pizzaStore.$state);
  await router.push({ name: "cart" });
  resetPizza();
};

const resetPizza = () => {
  pizzaStore.setName("");
  if (dataStore.isDataLoaded) {
    pizzaStore.setDough(dataStore.doughs[0].id);
    pizzaStore.setSize(dataStore.sizes[0].id);
    pizzaStore.setSauce(dataStore.sauces[0].id);
  }
  pizzaStore.setIngredients([]);
};

</script>


<style lang="scss">
@import "@/assets/scss/ds-system/ds";
@import "@/assets/scss/mixins/mixins";

.content__ingredients {
  width: 527px;
  margin-top: 15px;
  margin-right: auto;
  margin-bottom: 15px;
}

.content__pizza {
  width: 373px;
  margin-top: 15px;
  margin-bottom: 15px;
}

.content__result {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 25px;

  p {
    @include b-s24-h28;
    margin: 0;
  }

  button {
    margin-left: 12px;
    padding: 16px 45px;
  }
}
</style>
