<template>
  <div class="container mt-5">
    <div id="map" style="height: 400px"></div>
    <button @click="getCurrentLocation" class="button-14">Get Current Location</button>
    <input
      v-model="locationName"
      @keyup.enter="searchLocation"
      class="input-14"
      type="text"
      placeholder="Enter a location"
    />
    <button @click="searchLocation" class="button-14">Search</button>

    <h4>Timezone in selected location:</h4>
    <p>{{ timeZone }}</p>

    <h4>Local time in selected location:</h4>
    <p>{{ localTime }}</p>

    <button @click="deleteSelected" class="button-14">Delete Selected</button>
    <button @click="deleteAll" class="button-14">Delete All</button>
    <table class="table">
      <thead>
        <tr>
          <th>Searched</th>
          <th>Location History</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(loc, index) in paginatedLocations" :key="index">
          <td><input v-model="loc.selected" type="checkbox" /></td>
          <td>{{ loc.name }}</td>
        </tr>
      </tbody>
    </table>
    <div class="pagination">
      <button @click="currentPage--" :disabled="currentPage === 1" class="button-14">Prev</button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button @click="currentPage++" :disabled="currentPage === totalPages" class="button-14">
        Next
      </button>
    </div>
  </div>
</template>

<script>
import { toRaw } from 'vue'
export default {
  data() {
    return {
      map: null,
      locationName: '',
      locations: [],
      markers: [],
      currentPage: 1,

      timeZone: '',
      localTime: ''
    }
  },
  mounted() {
    window.onload = () => {
      this.initializeMap()
    }
  },
  computed: {
    totalPages() {
      return Math.ceil(this.locations.length / 10)
    },
    paginatedLocations() {
      const start = (this.currentPage - 1) * 10
      const end = start + 10
      return this.locations.slice(start, end)
    }
  },
  methods: {
    //initialize the google map;
    initializeMap() {
      const myLatLng = { lat: -25.363, lng: 131.044 }
      this.map = new google.maps.Map(document.getElementById('map'), {
        zoom: 4,
        center: myLatLng
      })
    },
    //get the user's current location;
    getCurrentLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          const pos = {
            lat: position.coords.latitude,
            lng: position.coords.longitude
          }
          this.addLocation(pos, 'Current Location')
        })
      } else {
        alert('Geolocation is not supported by this browser.')
      }
    },
    //get the location on the map based on user's input, it will pop up an alert if the input place is invalid;
    searchLocation() {
      const geocoder = new google.maps.Geocoder()
      geocoder.geocode({ address: this.locationName }, (results, status) => {
        if (status == 'OK') {
          //console.log(results)
          const pos = results[0].geometry.location
          this.addLocation({ lat: pos.lat(), lng: pos.lng() }, this.locationName)
          this.locationName = ''
        } else {
          alert('Something wrong with geocode')
        }
      })
    },
    //add the input location into the location array;
    //add a corresponding marker on the map;
    //add the marker into the markers array;
    addLocation(pos, name) {
      const marker = new google.maps.Marker({
        position: pos,
        map: this.map,
        title: name
      })
      if (marker.getMap() === null) {
        console.error('Marker was not added to map')
      }
      marker.setMap(this.map)
      this.markers.push(marker)
      this.locations.push({ name, lat: pos.lat, lng: pos.lng, selected: false })
      this.map.setCenter(pos)
      this.fetchTimeZone(pos.lat, pos.lng)
    },
    //get the local timezone based on the user's input location;
    async fetchTimeZone(lat, lng) {
      const timestamp = Math.round(new Date().getTime() / 1000)
      const apiUrl = `https://maps.googleapis.com/maps/api/timezone/json?location=${lat},${lng}&timestamp=${timestamp}&key=REPLACE_KEY_HERE`

      try {
        const response = await fetch(apiUrl)
        const data = await response.json()

        if (data.status === 'OK') {
          this.timeZone = data.timeZoneId
          this.displayLocalTime(data.timeZoneId)
        } else {
          console.error('Error fetching timezone:', data.errorMessage)
        }
      } catch (error) {
        console.error('Error:', error)
      }
    },
    //get the local time based on the user's input location;
    displayLocalTime(timeZoneId) {
      const localTime = new Intl.DateTimeFormat('en-US', {
        timeZone: timeZoneId,
        timeStyle: 'medium',
        dateStyle: 'medium'
      }).format(new Date())

      this.localTime = localTime
    },
    //remove markers from the map based on the selected want-to-delete locations;
    //update markers and locations array;
    deleteSelected() {
      for (let i = this.markers.length - 1; i >= 0; i--) {
        console.log('markers before deleting', this.markers)
        if (this.locations[i].selected) {
          const marker = this.markers[i]
          toRaw(marker).setMap(null)
          // console.log(this.markers)
          this.markers.splice(i, 1)
          console.log('markers after deleting', this.markers)
          this.locations.splice(i, 1)
        }
      }
    },
    // delete all markers and locations on the map;
    deleteAll() {
      for (let i = 0; i < this.markers.length; i++) {
        const marker = this.markers[i]
        toRaw(marker).setMap(null)
      }
      this.markers = []
      this.locations = []
    }
  }
}
</script>

<style>
/* CSS */
.button-14 {
  background-image: linear-gradient(#f7f8fa, #e7e9ec);
  border-color: #adb1b8 #a2a6ac #8d9096;
  border-style: solid;
  border-width: 1px;
  border-radius: 3px;
  box-shadow: rgba(255, 255, 255, 0.6) 0 1px 0 inset;
  box-sizing: border-box;
  color: #0f1111;
  cursor: pointer;
  display: inline-block;
  font-family: 'Amazon Ember', Arial, sans-serif;
  font-size: 14px;
  height: 29px;
  font-size: 13px;
  outline: 0;
  overflow: hidden;
  padding: 0 11px;
  text-align: center;
  text-decoration: none;
  text-overflow: ellipsis;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
}

.button-14:active {
  border-bottom-color: #a2a6ac;
}

.button-14:active:hover {
  border-bottom-color: #a2a6ac;
}

.button-14:hover {
  border-color: #a2a6ac #979aa1 #82858a;
}

.button-14:focus {
  border-color: #e77600;
  box-shadow: rgba(228, 121, 17, 0.5) 0 0 3px 2px;
  outline: 0;
}
.input-14 {
  padding: 0 11px;
  border-style: solid;
  border-width: 1px;
  border-radius: 3px;

  background-color: #ffffff;
  font-size: 14px;

  font-size: 13px;
  transition: all 0.2s ease-in-out 0s;

  font-weight: normal;
  height: 29px;
}
.input-14:focus {
  outline: none;
  border: 1px solid #007c89;
  box-shadow: inset 0 0 0 1px #007c89;
}
</style>
