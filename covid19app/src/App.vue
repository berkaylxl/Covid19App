<template>
  <div class="container mt-5">
    <h1 style="color: white">Covid Stastitics</h1>
    <div class="form-group mt-5">
      <select
        v-model="selectedCountry"
        class="form-control mb-5"
        @change="getDataByCountry"
      >
        <option value="Global">Global</option>
        <option
          v-for="country in countries"
          :value="country.Slug"
          :key="country"
        >
          {{ country.Country }}
        </option>
      </select>
    </div>
    <div class="row justify-content-around">
      <div class="col-md-4" style="background-color: #f39c12">
        <h5>Confirmed</h5>
        <h2>{{ numberFormat(confirmed) }}</h2>
      </div>
      <div class="col-md-4" style="background-color: #e74c3c">
        <h5>Deaths</h5>
        <h2>{{ numberFormat(deaths) }}</h2>
      </div>
      <div class="col-md-4" style="background-color: #2ecc71">
        <h5>Recovered</h5>
        <h2>{{ numberFormat(recovered) }}</h2>
      </div>
    </div>
    <hr />
    <table  class="table table-hover mt-5">
      <thead class="thead-light">
        <tr>
          <th scope="col">Date</th>
          <th scope="col">Confirmed</th>
          <th scope="col">Deaths</th>
          <th scope="col">Recovered</th>
        </tr>
      </thead>
      <tbody v-if="selectedCountry!=='Global'">
        <tr v-for="item in dailyData" :key="item">
          <td>{{ formatDate(item.Date)}}</td>
          <td>{{  numberFormat(item.Confirmed) }}</td>
          <td>{{  numberFormat(item.Deaths )}}</td>
          <td>{{  numberFormat(item.Recovered) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      countries: {},
      selectedCountry: "",
      confirmed: "",
      deaths: "",
      recovered: "",
      data: {},
      dailyData:{}
    };
  },
  methods: {
    numberFormat(number) {
      return number.toLocaleString();
    },
    formatDate(dateString){
      const date=new Date(dateString)
      return new Intl.DateTimeFormat('default',{dateStyle:'long'}).format(date)
    },
    getCountries() {
      axios.get("https://api.covid19api.com/countries").then((response) => {
        this.countries = response.data;
      });
    },
    getGlobalSummary() {
      axios.get("https://api.covid19api.com/summary").then((response) => {
        this.data = response.data;
        this.confirmed = this.data.Global.TotalConfirmed;
        this.deaths = this.data.Global.TotalDeaths;
        this.recovered = this.data.Global.TotalRecovered;
      });
    },
    getDataByCountry() {
      if (this.selectedCountry == "Global") {
        this.getGlobalSummary();
      } else {
        let data = this.data.Countries.filter((country) => {
          return country.Slug == this.selectedCountry;
        });
        data = data[0];
        this.confirmed = data.TotalConfirmed;
        this.deaths = data.TotalDeaths;
        this.recovered = data.TotalRecovered;
         this.getDailyDataByCountry()
      }
     
    },
    getDailyDataByCountry(){
      axios.get(`https://api.covid19api.com/country/${this.selectedCountry}?from=2020-03-01&to=${Date.now()}`)
      .then(response=>{
       this.dailyData=response.data
       
      })
    }
  },
  mounted() {
    this.getCountries();
    this.getGlobalSummary();
  },
};
</script>

<style>
body {
  background-color: black;
}
#app {
  text-align: center;
}
.col-md-4 {
  margin-top: 20px;
  box-shadow: 1px 10px 50px rgb(180, 13, 13);
}

select {
  box-shadow: 1px 1px 25px 5px rgb(5, 41, 245);
}
th,
td {
  color: white;
  font-weight: bold;
}
</style>