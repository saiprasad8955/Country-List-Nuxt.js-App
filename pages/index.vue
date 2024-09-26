<template>
  <div class="">
    <div class="content mx-auto">
      <h1 class="text-4xl font-bold my-2">Countries List</h1>

      <!-- Show error message if data fails to load -->
      <div v-if="error" class="error-message">
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

      <!-- <div class="p-2 font-bold">
       Total Countries: {{ countries.length }}
      </div> -->

      <!-- Country List -->
      <ul v-if="!error" class="country-list">
        <li
          v-for="country in filteredCountries"
          :key="country.alpha3Code"
          class="country-item"
        >
          <div class="flag-wrapper">
            <img :src="country.flags.png" alt="Country Flag" />
          </div>

          <!-- {{ country }} -->
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
              <button @click="showMap(country)" class="w-[100%]">
                Show Map
              </button>
              <button
                @click="navigateTo(`/country/${country.name.common}`)"
                class="w-[100%]"
              >
                Details
              </button>
            </div>
          </div>
        </li>
      </ul>

      <!-- Loading message while fetching data -->
      <div v-if="loading && !error">
        <p>Loading countries...</p>
      </div>
    </div>
  </div>
</template>

<script>
import { timezoneMap } from "../utils/constants";
import { DateTime } from "luxon";

export default {
  setup() {
    useHead({
      title: "Countries List",
      meta: [
        {
          name: "description",
          content: "A list of countries with their currency and timezone",
        },
      ],
    });
  },
  data() {
    return {
      countries: [],
      searchQuery: "",
      loading: true,
      error: false,
      errorMessage: "",
    };
  },
  async created() {
    try {
      const { data } = await useFetch("https://restcountries.com/v3.1/all");
      const formattedCountries = data.value
        .map((country) => {
          // Ensure currencies is defined before accessing
          const currencies = country.currencies
            ? Object.values(country.currencies)
            : [];
          const currencyName =
            currencies.length > 0 ? currencies[0].name : "Not available"; // Get the currency name if available

          return {
            ...country,
            currencyName,
          };
        })
        .sort((a, b) => {
          // Sort by name.common
          if (a.name.common < b.name.common) return -1;
          if (a.name.common > b.name.common) return 1;
          return 0;
        });

      this.countries = formattedCountries; // useFetch response is reactive, so use `.value`
    } catch (error) {
      // Handle error if API request fails
      console.error("Error fetching countries:", error);
      this.error = true;
      this.errorMessage =
        "Failed to load country data. Please try again later.";
    } finally {
      this.loading = false; // Set loading to false once request is complete
    }
  },
  computed: {
    // Filter countries based on search query
    filteredCountries() {
      return this.countries.filter((country) =>
        country.name.common
          .toLowerCase()
          .includes(this.searchQuery.toLowerCase())
      );
    },
  },
  methods: {
    // Show Google Maps with country name
    showMap(country) {
      const mapUrl = `https://www.google.com/maps/search/?api=1&query=${country.name.common}`;
      window.open(mapUrl, "_blank");
    },

    // Get the country specific date and time from timezones using luxon
    getCountryDateTime(timezone) {
      if (!timezone) return "Timezone not available";

      // If the provided timezone is a valid IANA name, use it
      const validTimezone = timezoneMap[timezone] || timezone;

      try {
        const dateTime = DateTime.now().setZone(validTimezone);
        return dateTime.toFormat("d 'of' MMMM yyyy, HH:mm");
      } catch (error) {
        return "Invalid Timezone";
      }
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.content {
  width: 50%;
}

.search-bar {
  padding: 10px;
  font-size: 16px;
}

.country-list {
  list-style-type: none;
  padding: 0;
}

.country-item {
  display: flex;
  align-items: center;
  gap: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 15px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.country-item img {
  width: 100%;
  height: auto;
  margin-right: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.country-info {
  flex-grow: 1;
}

.country-info h3 {
  margin: 0;
  font-size: 1.2rem;
}

.country-info p {
  margin: 5px 0;
  font-size: 0.9rem;
  color: #555;
}

.country-item button {
  background-color: #fff;
  border: 2px solid #1976d2;
  color: #1976d2;
  font-weight: bold;
  padding: 5px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.country-item button:hover {
  background-color: #1976d2;
  color: #fff;
}

.button-group {
  display: flex;
  gap: 10px;
}

.flag-wrapper {
  width: 350px;
  aspect-ratio: 16 / 11;
  overflow: hidden;
}

.flag-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.error-message {
  color: red;
  font-weight: bold;
  padding: 10px;
}
</style>
