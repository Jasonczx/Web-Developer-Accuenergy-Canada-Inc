<!-- Web.vue -->
<template>
  <div class="app">
    <h1>Location Search</h1>
    <div class="search">
      <input
        v-model="searchQuery"
        @keyup.enter="searchLocation"
        placeholder="Enter a location"
      />
      <button @click="searchLocation">Search</button>
    </div>
    <div class="map-container">
      <div ref="map" class="map"></div>
    </div>
    <div class="table">
      <table>
        <thead>
          <tr>
            <th>
              <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" />
            </th>
            <th>Location</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="location in locations" :key="location.id">
            <td>
              <input type="checkbox" v-model="location.selected" />
            </td>
            <td>{{ location.name }}</td>
          </tr>
        </tbody>
      </table>
      <button @click="deleteSelectedLocations">Delete Selected</button>
    </div>
    <div class="time-zone">
      <h2>Latest Searched Location</h2>
      <p>Time Zone: {{ latestLocation.timeZone }}</p>
      <p>Local Time: {{ latestLocation.localTime }}</p>
    </div>
  </div>
</template>

<script>
import { ref, reactive, watch } from 'vue';
import { getLocalTime, getTimeZone } from './api'; // Assuming you have API functions to fetch time zone and local time

export default {
  name: 'App',
  setup() {
    const map = ref(null);
    const locations = reactive([]);
    const searchQuery = ref('');
    const selectAll = ref(false);
    const latestLocation = reactive({
      timeZone: '',
      localTime: '',
    });

    // Function to get the current location from the browser
    const getCurrentLocation = () => {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          position => {
            const { latitude, longitude } = position.coords;
            const name = 'Current Location';
            const id = `current-${latitude}-${longitude}`;
            addLocation({ id, name, latitude, longitude });
          },
          error => {
            console.error('Error getting current location:', error);
          }
        );
      } else {
        console.error('Geolocation is not supported by this browser.');
      }
    };

    // Function to search for a location based on the input query
    const searchLocation = () => {
      const query = searchQuery.value;
      if (query.trim() !== '') {
        // Assuming you have an API function to search for locations
        // Replace 'searchLocations' with your actual API call
        searchLocations(query).then(locations => {
          locations.forEach(location => {
            const { id, name, latitude, longitude } = location;
            addLocation({ id, name, latitude, longitude });
          });
        });
      }
    };

    // Function to add a location to the map and locations list
    const addLocation = location => {
      const marker = new google.maps.Marker({
        position: { lat: location.latitude, lng: location.longitude },
        map: map.value,
      });
      locations.push({ ...location, marker, selected: false });
      updateLatestLocation(location);
    };

    // Function to delete all selected locations
    const deleteSelectedLocations = () => {
      const selectedLocations = locations.filter(location => location.selected);
      selectedLocations.forEach(location => {
        location.marker.setMap(null);
        locations.splice(locations.indexOf(location), 1);
      });
      selectAll.value = false;
    };

    // Function to update the latest searched location's time zone and local time
    const updateLatestLocation = location => {
      getTimeZone(location.latitude, location.longitude).then(timeZone => {
        latestLocation.timeZone = timeZone;
      });

      getLocalTime(location.latitude, location.longitude).then(localTime => {
        latestLocation.localTime = localTime;
      });
    };

    // Function to toggle select all checkbox
    const toggleSelectAll = () => {
      locations.forEach(location => {
        location.selected = selectAll.value;
      });
    };

    // Watch for changes in the selectAll value to update individual location selections
    watch(selectAll, newValue => {
      locations.forEach(location => {
        location.selected = newValue;
      });
    });

    return {
      map,
      locations,
      searchQuery,
      selectAll,
      latestLocation,
      getCurrentLocation,
      searchLocation,
      deleteSelectedLocations,
    };
  },
  mounted() {
    // Initialize the map
    const mapOptions = {
      center: { lat: 0, lng: 0 },
      zoom: 1,
    };
    this.map = new google.maps.Map(this.$refs.map, mapOptions);
  },
};
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 2rem;
}

.search {
  display: flex;
  margin-bottom: 1rem;
}

.search input {
  padding: 0.5rem;
}

.search button {
  margin-left: 1rem;
  padding: 0.5rem 1rem;
}

.map-container {
  width: 100%;
  height: 300px;
  margin-bottom: 1rem;
}

.table {
  width: 100%;
  margin-bottom: 1rem;
}

.table table {
  width: 100%;
  border-collapse: collapse;
}

.table th,
.table td {
  padding: 0.5rem;
  border: 1px solid #ccc;
}

.table th {
  text-align: left;
}

.time-zone {
  text-align: center;
}
</style>
