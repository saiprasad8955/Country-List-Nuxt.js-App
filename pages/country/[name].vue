<template>
  <div class="flex justify-center w-[100%]">
    <div v-if="loading" class="text-center py-8">
      <p class="text-xl font-bold">Loading...</p>
    </div>

    <div v-else-if="country" class="w-[70%] p-10">
      <button
        @click="$router.push('/')"
        class="px-4 py-2 mb-8 rounded-md bg-blue-500 text-white hover:bg-blue-700"
      >
        Back
      </button>

      <h1 class="text-5xl font-bold mb-4">{{ country.name.common }}</h1>

      <div class="flex flex-row justify-start gap-[50px] h-[320px] mb-10">
        <!-- Displayed image  -->
        <!-- 
        /> -->
        <div class="aspect-w-3 aspect-h-3">
          <img
            :src="country.flags.png"
            alt="Country Flag"
            class="h-full w-full rounded-lg shadow-md"
          />
        </div>

        <!-- Display all other details  -->
        <div class="details flex flex-col justify-between">
          <p class="text-gray-700">
            <strong>Native Name: </strong>
            {{
              Object.values(country.name.nativeName)
                .map((cntry) => cntry.common)
                .join(", ")
            }}
          </p>

          <p class="text-gray-700">
            <strong>Capital: </strong> {{ country.capital[0] }}
          </p>
          <p class="text-gray-700">
            <strong>Population: </strong> {{ country.population }}
          </p>
          <p class="text-gray-700">
            <strong>Region: </strong> {{ country.region }}
          </p>
          <p class="text-gray-700">
            <strong>Subregion: </strong> {{ country.subregion }}
          </p>
          <p class="text-gray-700">
            <strong>Area: </strong> {{ country.area }} km²
          </p>
          <p class="text-gray-700">
            <strong>Languages: </strong>
            {{ Object.values(country.languages).join(",") }}
          </p>
          <p class="text-gray-700">
            <strong>Timezones: </strong> {{ country.timezones.join(", ") }}
          </p>

          <p class="text-gray-700">
            <strong>Currencies: </strong>
            <span v-if="country.currencies && country.currencies">
              {{
                Object.values(country.currencies)
                  .map((obj) => obj.name)
                  .join(", ")
              }}
            </span>
            <span v-else>No currencies available</span>
          </p>
        </div>
      </div>

      <div class="neighbour-countries border border-black p-5">
        <div class="mb-8">
          <h1 class="text-3xl font-bold">Neighbour Countries</h1>
        </div>
        <div class="grid grid-cols-2 md:grid-cols-3 gap-10">
          <div
            v-for="flag in neighbourCountries"
            :key="flag"
            class="flex flex-col items-center justify-center aspect-[3/2] object-contain"
          >
            <img
              :src="flag"
              :alt="`Flag of ${country.name}`"
              class="object-cover h-full w-full rounded-lg mb-2 border border-gray-400"
            />
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <p class="text-center text-xl font-bold">Country not found.</p>
    </div>
  </div>
</template>
<script>
import { ref } from "vue";
import { useRoute } from "vue-router";

export default {
  setup() {
    const country = ref(null); // Reactive country property
    const loading = ref(true); // Loading state
    const neighbourCountries = ref([]); // Neighbour countries
    const route = useRoute();

    const fetchCountry = async () => {
      loading.value = true; // Set loading to true before fetching
      try {
        const response = await fetch(
          `https://restcountries.com/v3.1/name/${route.params.name}`
        );

        const countryJson = await response.json();
        const borders = countryJson[0].borders;
        country.value = countryJson[0]; // Set the country to the first object in the array

        if (borders) {
          const borderPromises = borders.map(async (border) => {
            const borderResponse = await fetch(
              `https://restcountries.com/v3.1/alpha/${border}`
            );

            if (!borderResponse.ok) {
              throw new Error(
                `Failed to fetch border country data: ${borderResponse.status}`
              );
            }

            // Clone the response to avoid consuming the stream
            const clonedResponse = borderResponse.clone();
            const borderData = await clonedResponse.json();

            return borderData[0].flags.png;
          });

          neighbourCountries.value = await Promise.all(borderPromises);
        }
      } catch (error) {
        console.error("Error fetching country details:", error);
      } finally {
        loading.value = false;
      }
    };

    fetchCountry();

    // Update the page title dynamically based on the country name
    watchEffect(() => {
      if (route.params.name) {
        useHead({
          title: `Country : ${route.params.name}`,
          description: "Country details page",
        });
      }
    });

    return {
      country,
      neighbourCountries,
      loading,
    };
  },
};
</script>