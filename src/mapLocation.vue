<template>
  <div class="container">
    <div class="row mx-md-n5 p-3">
      <div class="col-md-2"></div>
      <div class="col-md-2"></div>
      <button @click="getLocation" class="btn btn-primary col-md-2">
        current location
      </button>
    </div>
    <div class="row mx-md-n5 p-3">
      <div class="col-md-3"></div>
      <input
        class="col-md-2"
        type="text"
        name="address"
        id="address"
        placeholder="address"
        v-model="search"
        @keyup.enter="getGeocode"
      />
      <button class="btn btn-primary col-md-2" @click="getGeocode">
        search
      </button>
    </div>
    <GoogleMap
      ref="mapRef"
      api-key="AIzaSyBI2MvvfntJsB5akLrb9Mxy8FgrqBIP4QM"
      style="width: 100%; height: 500px"
      :center="center"
      :zoom="7"
    >
      <Marker
        v-for="(location, i) in locations"
        :options="{ position: location }"
        :key="i"
      />
    </GoogleMap>
    <div class="row">
    
    <div class="col-md-6">
    <table id="tableComponent" class="table table-bordered table-striped" >
      <thead>
        <tr>
          <th class="col-md-4 text-center" ><button class="btn btn-primary" @click="removeRows">Remove Selected</button></th>
          <th  class="col-md-6 text-center"> Location</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, i) in paginatedData" :key="item.index">
          <td class="text-center"><input type="checkbox" v-model="selectedRows" :value="i" /></td>
          <td class="text-center">{{ item }}</td>
        </tr>
      </tbody>
    </table>
    </div>
    </div>
    <button @click="backPage">prev</button>
    <button
      v-for="item in Math.ceil(places.length / perPage)"
      :key="item"
      @click="() => goToPage(item)"
    >
      {{ item }}
    </button>
    <button @click="nextPage">next</button>
    <div class="row">
      <div  class="col-md-4"></div>
      <div  class="col-md-6">
    <h2>Lasted location</h2>
    <p >Time Zone: {{ timeZone }}</p>
    <p >Local Time: {{ localTime }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref, toRaw } from "vue";
import { GoogleMap, Marker } from "vue3-google-map";

export default defineComponent({
  components: { GoogleMap, Marker },
  setup() {
    const center = ref({ lat: 0, lng: 0 });
    const locations = ref([]);
    const mapRef = ref(null);
    const search = ref(null);
    const places = ref([]);
    const paginatedData = ref(null);
    const perPage = 10;
    const page = ref(1);
    const selectedRows = ref([]);
    const timeZone = ref(null);
    const localTime = ref(null);
    return {
      center,
      mapRef,
      locations,
      search,
      paginatedData,
      perPage,
      page,
      places,
      selectedRows,
      timeZone,
      localTime,
    };
  },
  methods: {
    getLocation() {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          this.center.lat = position.coords.latitude;
          this.center.lng = position.coords.longitude;
          this.$refs.mapRef.map.panTo({
            lat: this.center.lat,
            lng: this.center.lng,
          });
          this.locations.push({ lat: this.center.lat, lng: this.center.lng });
          this.handlePagination();
        },
        (error) => {
          console.log(error.message);
        }
      );
    },
    async getGeocode() {
      const address = this.search;
      this.places.push(address);
      await fetch(
        `https://maps.googleapis.com/maps/api/geocode/json?address=${address}&key=AIzaSyBI2MvvfntJsB5akLrb9Mxy8FgrqBIP4QM`
      )
        .then((response) => {
          return response.json();
        })
        .then((jsonData) => {
          this.locations.push(jsonData.results[0].geometry.location);
          this.$refs.mapRef.map.panTo(jsonData.results[0].geometry.location);
          this.handlePagination();
        })
        .catch((error) => {
          console.log(error);
        });
      this.showTimeZone();
    },
    handlePagination() {
      this.paginatedData = this.places.slice(
        (this.page - 1) * this.perPage,
        this.page * this.perPage
      );
      this.selectedRows = [];
    },
    nextPage() {
      if (this.page !== Math.ceil(this.places.length / this.perPage)) {
        this.page += 1;
        this.handlePagination();
      }
    },
    backPage() {
      if (this.page !== 1) {
        this.page -= 1;
        this.handlePagination();
      }
    },
    goToPage(numPage) {
      this.page = numPage;
      this.handlePagination();
    },
    removeRows() {
      const Rows = this.selectedRows.map((row) => row + (this.page - 1) * 10);
      this.places = this.places.filter(
        (place) => !Rows.includes(this.places.indexOf(place))
      );
      this.locations = this.locations.filter(
        (location) => !Rows.includes(this.locations.indexOf(location))
      );
      console.log(this.locations);

      this.handlePagination();
    },
    async showTimeZone() {
      const { lat, lng } = toRaw(this.locations).pop();
      var targetDate = new Date();
      var timestamp =
        targetDate.getTime() / 1000 + targetDate.getTimezoneOffset() * 60;
      await fetch(
        `https://maps.googleapis.com/maps/api/timezone/json?location=${lat}%2C${lng}&timestamp=
${timestamp}&language=en&key=AIzaSyBI2MvvfntJsB5akLrb9Mxy8FgrqBIP4QM`
      )
        .then((response) => {
          return response.json();
        })
        .then((jsonData) => {
          this.timeZone = jsonData.timeZoneId;
          var offsets = jsonData.dstOffset * 1000 + jsonData.rawOffset * 1000;
          this.localTime = new Date(timestamp * 1000 + offsets);
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
});
</script>
