<template>
  <div>
    <li
      class="country-item flex items-center gap-5 border border-gray-300 rounded-lg p-4 mb-4 shadow-md"
    >

      <div class="w-[350px] aspect-[16/11] overflow-hidden">
        <img
          :src="country.flags.png"
          alt="Country Flag"
          class="w-full h-full object-cover border border-gray-400"
        />
      </div>

      <div class="w-[100%] flex flex-col gap-3">
        <!-- Display the country name  -->
        <h1 class="text-2xl font-extrabold">{{ country.name.common }}</h1>

        <p class="font-bold" v-if="country.currencyName">
          Currency: {{ country.currencyName }}
        </p>
        <p class="font-bold" v-else>Currency: Not Available</p>

        <!-- Display the current date and time in the country's time zone with a safe check -->
        <p
          class="font-bold"
          v-if="country.timezones && country.timezones.length > 0"
        >
          Current date and time:
          {{ getCountryDateTime(country.timezones[0]) }}
        </p>
        <p class="font-bold" v-else>Timezone: Not Available</p>

        <div class="flex justify-between gap-5">
          <!-- Show Map Button -->
          <button @click="showMap(country)" class="btn-primary">
            Show Map
          </button>
          <button
            @click="navigateTo(`/country/${country.name.common}`)"
            class="btn-primary"
          >
            Details
          </button>
        </div>
      </div>
    </li>
  </div>
</template>

<script setup >
import { DateTime } from "luxon";
import { timezoneMap } from "../utils/constants";

const props = defineProps({
  country: Object,
});

// Method to show map using Google Maps API
function showMap(country) {
  const mapUrl = `https://www.google.com/maps/search/?api=1&query=${country.name.common}`;
  window.open(mapUrl, "_blank");
}

// Method to get current date and time for the country's timezone
function getCountryDateTime(timezone) {
  if (!timezone) return "Timezone not available";

  // If the provided timezone is a valid IANA name, use it
  const validTimezone = timezoneMap[timezone] || timezone;

  try {
    const dateTime = DateTime.now().setZone(validTimezone);
    return dateTime.toFormat("d 'of' MMMM yyyy, HH:mm");
  } catch (error) {
    return "Invalid Timezone";
  }
}

// Method to get currency name
function getCurrencyName(currencies) {
  const currencyKey = Object.keys(currencies)[0];
  return currencies[currencyKey].name;
}

// Navigation to country details using Nuxt's navigateTo
function navigateToCountry(countryName) {
  navigateTo(`/country/${countryName}`);
}
</script>

<style scoped>
.btn-primary {
  @apply bg-white w-full border-2 border-blue-700 text-blue-700 font-bold p-2 rounded cursor-pointer transition duration-300 ease-in-out hover:bg-blue-700 hover:text-white;
}
</style>