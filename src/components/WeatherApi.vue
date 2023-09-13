<template>
  <div class="centered-card" :class="[enteredCondition]">
    <v-card class="mx-auto">
      <v-card-item v-if="data.location">
        {{ data.location.name }}, {{ data.location.country }}
        <div
          v-for="(input, index) in inputs"
          :key="index"
          class="input_container"
        >
          <input
            :type="input.type"
            :placeholder="input.placeholder"
            style="border: 1px solid black"
            v-model="input.value"
            @keyup.enter="handleInputEnter(input.enterHandler)"
          />
        </div>
      </v-card-item>
      <v-card-text class="py-0">
        <v-row align="center" no-gutters>
          <v-col class="text-h2" cols="6" v-if="data.current">
            {{ data.current.temp_c }}&deg;C
          </v-col>
          <animation-front :path="path"> </animation-front>
        </v-row>
      </v-card-text>

      <div class="d-flex py-3 justify-space-between">
        <v-list-item density="compact" prepend-icon="mdi-weather-windy">
          <v-list-item-subtitle color="black" v-if="data.current"
            >{{ data.current.wind_kph }} km/h</v-list-item-subtitle
          >
        </v-list-item>

        <v-list-item density="compact" prepend-icon="mdi-weather-rainy">
          <v-list-item-subtitle v-if="data.current"
            >{{ data.current.humidity }}%</v-list-item-subtitle
          >
        </v-list-item>

        <v-list-item density="compact">
          <v-list-item-subtitle color="black" v-if="data.current"
            >{{ data.location.localtime }}
          </v-list-item-subtitle>
        </v-list-item>
      </div>
    </v-card>
  </div>
</template>

<script>
import axios from "axios";
import AnimationFront from "./AnimationFront.vue";
export default {
  components: {
    AnimationFront,
  },
  data: () => ({
    data: {},
    country: null,
    longitude: null,
    latitude: null,
    enteredCondition: "",
    inputs: [
      {
        type: "string",
        placeholder: "City or country",
        value: null,
        model: "this.country",
        enterHandler: "getAPI",
      },
      {
        type: "number",
        placeholder: "Latitude",
        value: null,
        model: "this.latitude",
        enterHandler: "getAPILocation",
      },
      {
        type: "number",
        placeholder: "Longitude",
        value: null,
        model: "this.longitude",
        enterHandler: "getAPILocation",
      },
    ],
  }),
  computed: {
    path() {
      console.log(this.enteredCondition);
      if (this.enteredCondition === "sunny") {
        return "animation_sun.json";
      } else if (this.enteredCondition === "overcast") {
        return "animation_cloudy.json";
      } else if (this.enteredCondition === "cloudy") {
        return "animation_cloudy.json";
      } else if (this.enteredCondition === "light rain") {
        return "animation_rain.json";
      } else if (this.enteredCondition === "moderate rain") {
        return "animation_rain.json";
      } else if (
        this.enteredCondition === "moderate or heavy rain with thunder"
      ) {
        return "animation_rain.json";
      } else if (this.enteredCondition === "moderate rain at times") {
        return "animation_rain.json";
      } else if (this.enteredCondition === "thundery outbreaks possible") {
        return "animation_rain.json";
      } else if (this.enteredCondition === "clear") {
        return "animation_moon.json";
      } else if (this.enteredCondition === "partly cloudy") {
        return "animation_partly_cloudy.json";
      } else if (this.enteredCondition === "heavy snow") {
        return "animation_snow.json";
      } else if (this.enteredCondition === "fog") {
        return "animation_fog.json";
      } else if (this.enteredCondition === "mist") {
        return "animation_fog.json";
      }
    },
  },
  methods: {
    getAPI() {
      return axios
        .get(
          `http://api.weatherapi.com/v1/current.json?key=5b38600981ec4ccba1491803230807&q=${this.country}&aqi=yes`
        )
        .then((response) => {
          this.data = response.data;
          console.log(response.data);
          this.enteredCondition =
            this.data.current.condition.text.toLowerCase();
          this.country = null;
          this.latitude = response.data.location.lat;
          this.longitude = response.data.location.lon;
        })
        .catch((error) => {
          alert(`${this.country} not found`);
          this.enteredCondition = "";
        });
    },
    getAPILocation() {
      return axios
        .get(
          `http://api.weatherapi.com/v1/current.json?key=5b38600981ec4ccba1491803230807&q=${this.latitude},${this.longitude}&aqi=yes`
        )
        .then((response) => {
          this.country = response.data.location.name;
          this.getAPI();
          if (this.country) {
            return this.latitude == null, this.longitude == null;
          }
        });
    },
    handleInputEnter(handler) {
      if (handler === "getAPI") {
        if (this.inputs[0].value) {
          this.country = this.inputs[0].value;
          this.latitude = null;
          this.longitude = null;
          this.getAPI();
        }
      } else if (handler === "getAPILocation") {
        if (this.inputs[1].value && this.inputs[2].value) {
          this.latitude = this.inputs[1].value;
          this.longitude = this.inputs[2].value;
          this.country = null;
          this.getAPILocation();
        }
      }
    },
  },
  mounted() {
    navigator.geolocation.getCurrentPosition((data) => {
      this.latitude = data.coords.latitude;
      this.longitude = data.coords.longitude;
      this.getAPILocation();
    });
  },
};
</script>

<style scoped>
.centered-card {
  background: rgb(255, 255, 255);
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  transition: background-color 1.5s ease;
}
.input_container {
  margin-bottom: 10px;
}
.v-card {
  background: rgb(148, 166, 188);
  width: 550px;
  height: 350px;
}
.sunny {
  background: rgba(225, 255, 0, 0.557);
}
.cloudy {
  background: rgba(32, 141, 147, 0.401);
}
.fog {
  background: rgba(53, 55, 56, 0.655);
}
.partly_cloudy {
  background: rgb(137, 151, 255);
}
.overcast {
  background: rgba(45, 84, 132, 0.749);
}
.rain {
  background: rgba(29, 45, 85, 0.49);
}
.clear {
  background: rgb(29, 26, 37);
}
.mist {
  background: rgba(28, 45, 59, 0.64);
}
.snow {
  background: rgba(122, 216, 252, 0.492);
}
.patchy_rain_possible {
  background: rgb(104, 108, 163);
}
.heavy_rain {
  background: rgb(14, 18, 82);
}
</style>
