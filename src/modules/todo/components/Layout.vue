<template>
  <h2>{{ title1 }}</h2>
  <h3>
    {{ title2 }}
  </h3>
  <p class="buttons">
    <button @click="handleFetchData">Reload</button>
    <button @click="changeSystem">
      {{ systemUnit === "imperial" ? "to metric" : "to imperial" }}
    </button>
  </p>
  <hr />
  <slot></slot>
  <footer>
    <p class="buttons">
      <button @click="handleChangeDay(-1)" v-if="plusDay">← Previous Day</button
      ><span v-else></span
      ><button @click="handleChangeDay(+1)">Next Day →</button>
    </p>
  </footer>
</template>

<script>
import { format, addDays } from "date-fns";
import { computed } from "vue";

export default {
  name: "AsteroidLayout",
  props: ["data", "fetchData", "system", "plusDay", "changeDay"],
  setup({ fetchData, data, system, plusDay, changeDay }) {
    const title1 = computed(
      () => `${format(addDays(new Date(), plusDay || 0), "EEEE d-MMM")},`
    );
    const title2 = computed(
      () => `There Will Be
      ${data?.length} Near Misses`
    );

    const systemUnit = computed(() => system);

    const handleFetchData = () => fetchData();

    const changeSystem = () => {
      systemUnit.value === "imperial"
        ? fetchData("metric")
        : fetchData("imperial");
    };

    const handleChangeDay = (value) => {
      changeDay(value);
    };

    return {
      title1,
      title2,
      handleFetchData,
      systemUnit,
      changeSystem,
      handleChangeDay,
    };
  },
};
</script>

<style scoped>
.buttons {
  display: flex;
  justify-content: space-between;
}
button {
  margin: 0;
  padding: 0;
  border: none;
  font-size: 1em;
  cursor: pointer;
  font-family: Ubuntu Mono, Menlo, Consolas, Monaco, Liberation Mono,
    Lucida Console, monospace;
  font-weight: 600;
  background-color: transparent;
  text-transform: capitalize;
  color: var(--link);
}
button:hover {
  color: var(--primary);
}
.active {
  color: var(--secondary);
}
footer {
  color: var(--secondary);
  font-weight: 500;
  text-align: center;
}
</style>
