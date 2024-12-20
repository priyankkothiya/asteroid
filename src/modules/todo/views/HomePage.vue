<script setup lang="ts">
import Loading from "@/modules/todo/components/Loading.vue";
import Layout from "@/modules/todo/components/Layout.vue";
import Asteroid from "@/modules/todo/components/Asteroid.vue";
import { TwToggle, TwDialog } from "../src/build";
import { onMounted, ref, watch } from "vue";
import { addDays } from "date-fns";
import VueTailwindDatepicker from "vue-tailwind-datepicker";

const dark = ref(false);
let data = ref([]);
let isLoading = ref(false);
let system = ref("imperial");
let plusDay = ref(0);

onMounted(() => {
  fetchData(system.value);
});

const fetchData = (type: string) => {
  isLoading.value = true;
  system.value = type;
  fetch(
    `https://api.nasa.gov/neo/rest/v1/feed?end_date=${getDate()}&&api_key=jVNOPOZAhXTceEbTVTsfzrscTLC5ykmFHmVV6yW7`
  )
    .then((res) => res.json())
    .then((res) => {
      let fetchedData = res.near_earth_objects[getDate()]
        .sort(
          (a: any, b: any) =>
            a.close_approach_data[0].epoch_date_close_approach -
            b.close_approach_data[0].epoch_date_close_approach
        )
        .filter(
          (asteroid: any) =>
            asteroid.close_approach_data[0].epoch_date_close_approach >
            new Date()
        );

      const hazards = fetchedData.reduce((acc: number, curr: any) => {
        if (curr.is_potentially_hazardous_asteroid) {
          return acc + 1;
        }
        return acc;
      }, 0);
      hazards
        ? (document.title = `${hazards} Potential Hazards ⚠️ - Near Earth Asteroids`)
        : (document.title = "Near Earth Asteroids");

      data.value = fetchedData;
      isLoading.value = false;
    });
};

const getDate = () => {
  return `${addDays(new Date(), plusDay.value).toISOString().substr(0, 10)}`;
};

const changeDay = (day: number) => {
  plusDay.value += day;
  fetchData(system.value);
};
watch(dark, (value) => {
  if (value) {
    document.body.setAttribute("data-theme", "dark");
  } else {
    document.body.setAttribute("data-theme", "light");
  }
});

onMounted(async () => {
  const windowPreferenceDark =
    window.matchMedia &&
    window.matchMedia("(prefers-color-scheme: dark)").matches;

  if (windowPreferenceDark) {
    document.body.setAttribute("data-theme", "dark");
  } else {
    document.body.setAttribute("data-theme", "light");
  }
});
</script>

<template>
  <div class="container mx-auto max-w-4xl">
    <div class="grid md:grid-cols-3 gap-6">
      <div class="md:col-span-2 space-y-4" id="product-list">
        <Loading v-if="isLoading" />
            <Layout
              v-else
              :data="data"
              :fetchData="fetchData"
              :system="system"
              :plusDay="plusDay"
              :changeDay="changeDay"
            >
              <div v-for="asteroid in data" :key="asteroid.id">
                <Asteroid :asteroid="asteroid" :system="system" />
              </div>
            </Layout>
      </div>
    </div>
  </div>
</template>
