<template>
  <div class="mx-auto w-[50%]">
    <h1 class="text-4xl font-bold my-2">Countries List</h1>

    <!-- Show error message if data fails to load -->
    <div v-if="error" class="text-red-500 font-bold p-4">
      <p>Error: {{ errorMessage }}</p>
    </div>

    <!-- Search Bar -->
    <div class="relative">
      <input
        v-if="!error"
        type="text"
        v-model="searchQuery"
        placeholder="Search country by name.."
        class="my-2 p-2 border border-gray-300 rounded-md w-full"
      />
      <button
        type="submit"
        class="absolute inset-y-2 right-0 flex items-center justify-center px-3 cursor-default rounded-r-md bg-blue-500 text-white hover:bg-blue-600"
      >
        <svg
          class="h-5 w-5"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0 1 14 0z"
          ></path>
        </svg>
      </button>
    </div>

    <!-- Country List -->
    <ul v-if="!error">
      <CountryItem
        v-for="country in filteredCountries"
        :key="country.alpha3Code"
        :country="country"
      />
    </ul>

    <!-- Loading message while fetching data -->
    <div v-if="loading && !error">
      <p>Loading countries...</p>
    </div>
  </div>
</template>

<script setup>
import { DateTime } from "luxon";
import { timezoneMap } from "../utils/constants";

useHead({
  title: "Countries List",
  meta: [
    {
      name: "description",
      content: "A list of countries with their currencies",
    },
  ],
});

// Reactive references
const countries = ref([]);
const searchQuery = ref("");
const loading = ref(true);
const error = ref(false);
const errorMessage = ref("");

// Fetch data using useFetch in setup
onMounted(async () => {
  try {
    const data = await $fetch("https://restcountries.com/v3.1/all");
    const formattedCountries = data
      .map((country) => {
        const currencies = country.currencies
          ? Object.values(country.currencies)
          : [];
        const currencyName =
          currencies.length > 0 ? currencies[0].name : "Not available";
        return { ...country, currencyName };
      })
      .sort((a, b) => a.name.common.localeCompare(b.name.common));

    countries.value = formattedCountries;
  } catch (err) {
    console.error("Error fetching countries:", err);
    error.value = true;
    errorMessage.value = "Failed to load country data. Please try again later.";
  } finally {
    loading.value = false;
  }
});

// Computed property to filter countries based on the search query
const filteredCountries = computed(() => {
  return countries.value.filter((country) =>
    country.name.common.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});
</script>
