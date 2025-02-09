<template>
  <input
    v-model="enteredCity"
    @keyup="enterCity"
    @change="chooseCity"
    list="data-list"
    autocomplete="off"
    :placeholder="placeholder"
  />
  <datalist id="data-list">
    <option v-for="(option, key) in dataListOptions" :key="key" :value="option.longName"></option>
  </datalist>
</template>

<script setup>
import { ref } from 'vue'
import autocompleteCityNames from '@/assets/autocompleteCityNames'

const props = defineProps({
  cityName: String,
  placeholder: String
})
const emit = defineEmits(['selectCity'])

const enteredCity = ref(props.cityName || '')
const dataListOptions = ref([])

function enterCity(e) {
  /* Ignoring the keyup event if the user has entered less than two
  characters or the entered characters are arrow keys, Enter or
  Escape. Those keys, when used to navigate the <datalist>, will make
  the <datalist> glitch if we don't return here. */
  if (
    enteredCity.value.length < 2 ||
    ['ArrowUp', 'ArrowRight', 'ArrowDown', 'ArrowLeft', 'Enter', 'Escape'].includes(e.key)
  ) {
    return;
  }

  dataListOptions.value = [];
  // Ignoring everything after the first comma:
  enteredCity.value = enteredCity.value.replace(/,.*/, '').toLowerCase()

  for (const city of autocompleteCityNames.cities) {
    if (city[0].toLowerCase().startsWith(enteredCity.value)) {
      const parts = [city[0]]

      /* If the city is in a region, we'll add it. Not every city in
      the data belongs to a region. */
      if (autocompleteCityNames.regions[city[2] + '.' + city[1]]) {
        parts.push(autocompleteCityNames.regions[city[2] + '.' + city[1]])
      }

      parts.push(autocompleteCityNames.countries[city[2]])

      city.longName = parts.join(', ')
      dataListOptions.value.push(city)
    }
  }
}

/* Since there are no events that can be attached to a <datalist>,
the best way to detect when the user has selected a city is to use
the "change" event on the <input> element: */
function chooseCity() {
  for (const option of dataListOptions.value) {
    if (enteredCity.value === option.longName) {
      emit('selectCity', {
        name: option[0],
        longName: option.longName,
        regionId: option[1],
        countryId: option[2],
        latitude: option[3],
        longitude: option[4]
      })

      return
    }
  }

  /* In case the user has entered a few characters but NOT selected a
  city on the list, we'll reset the city: */
  enteredCity.value = ''
}
</script>
