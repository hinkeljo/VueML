<template>
  <v-container>
      <v-layout wrap>
        <v-flex sm12 md6 offset-md3>
          <v-card elevation="4" light tag="section">
            <v-card-title>
              <v-layout align-center justify-start>
                <h3>
                  {{ platformName }}
                </h3>
              </v-layout>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
              <p>Enter the lead data:</p>
              <v-form>  
                <v-text-field v-model="scoreSelect" type="number" :rules="[v => !!v || 'Score is required']" label="Lead Score" outlined required/>
                <v-text-field v-model="ageSelect" type="number" :rules="[v => !!v || 'Age is required']" label="Age" outlined required/>
                <v-select v-model="genderSelect" :items="genderItems" :rules="[v => !!v || 'Gender is required']" label="Gender" outlined required></v-select>
                <v-select v-model="countrySelect" :items="countryItems" :rules="[v => !!v || 'Country is required']" label="Country" outlined required></v-select>
                <v-select v-model="languageSelect" :items="languageItems" :rules="[v => !!v || 'Language is required']" label="Language" outlined required></v-select>
                <v-text-field v-model="sizeSelect" type="number" :rules="[v => !!v || 'Company Size is required']" label="Company Size" outlined required/>
                <v-select v-model="industrySelect" :items="industryItems" :rules="[v => !!v || 'Industry is required']" label="Industry" outlined required></v-select>
              </v-form>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-actions :class="{ 'pa-3': $vuetify.breakpoint.smAndUp }">
              <v-btn color="error" text @click="clear()">
                Clear
              </v-btn>
              <v-spacer></v-spacer>
              <v-btn v-if="isSetup" color="info" :large="$vuetify.breakpoint.smAndUp" @click="classify()">
                <v-icon left>mdi-cloud-upload</v-icon> Predict
              </v-btn>
              <v-btn v-else color="info" :large="$vuetify.breakpoint.smAndUp" disabled>
                Please wait...
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-flex>
        <v-flex sm12 md6 offset-md3>
          <v-layout align-end justify-end>
            <p class="caption my-3">Powered by <a href="https://ml5js.org/">ml5.js</a> and <a href="https://vuejs.org/">Vue.js</a></p>
          </v-layout>
        </v-flex>
      </v-layout>
      <v-dialog v-model="resultDialog" width="600">
        <v-card>
          <v-card-title class="text-h5 grey lighten-3" >
            Prediction results
          </v-card-title>
          <div id="chart">
            <apexchart type="bar" height="400" :options="chartOptions" :series="series" ref="resultChart"></apexchart>
          </div>
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="resultDialog = false">
              Close
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
</template>

<script>
  import VueApexCharts from 'vue-apexcharts'
  export default {
    name: 'MLMain',
    components: {
      'apexchart': 
      VueApexCharts,
    },
    data: () => ({
      isSetup: false,
      resultDialog: false,
      salesreps: ['Max', 'Erika', 'Andre', 'Frank'],
      neuralNetwork: null,
      ranking: [],
      platformName: 'Predict the best sales rep for your customers  ',
      scoreSelect: null,
      ageSelect: null,
      genderSelect: null,
      genderItems: ['male', 'female', 'diverse'],
      countrySelect: null,
      countryItems: ['Germany', 'France', 'Poland', 'England'],
      languageSelect: null,
      languageItems: ['German', 'French', 'Polish', 'Russian', 'English'],
      sizeSelect: null,
      industrySelect: null, 
      industryItems: ['Software', 'Marketing', 'Retail', 'Automotive', 'Machines', 'Fashion', 'Architecture'],
      series: [ { data: [] } ],
      chartOptions: {
        chart: {
          type: 'bar',
          height: 350
        },
        plotOptions: {
          bar: {
            borderRadius: 2,
            horizontal: true,
          }
        },
        dataLabels: {
          enabled: false
        },
        xaxis: {
          categories: [],
        }
      },
    }),
    computed: {
    },
    methods: {
      predict() {
        console.log('Predicting results...'); 
      },
      clear() {
        console.log('Clearing input...'); 
        this.scoreSelect = null; 
        this.ageSelect = null; 
        this.genderSelect = null;
        this.countrySelect = null; 
        this.languageSelect = null; 
        this.sizeSelect = null;
        this.industrySelect = null;
      },
      setupNeuralNetwork() {
        let nnOptions = {
          dataUrl: './data/trainingdata.csv',
          inputs: ['salesRep','leadScore','customerAge','customerGender','customerCountry','customerLanguage','companySize','companyIndustry'],
          outputs: ['result'],
          task: 'classification',
          debug: false
        };

        this.neuralNetwork = window.ml5.neuralNetwork(nnOptions, this.modelReady)
      }, 
      modelReady() {
        this.neuralNetwork.normalizeData();
        this.neuralNetwork.train({ epochs: 1 }, this.whileTraining, this.finishedTraining);
      }, 
      whileTraining(epoch, logs) {
        console.log(`Epoch: ${epoch} - loss: ${logs.loss.toFixed(2)}`);
      },
      finishedTraining() {
        console.log('Training completed!');
        this.isSetup = true; 
      },
      classify() {
        if(!this.isInputValid()) {
          alert('Please fill out all the fields!'); 
          return; 
        }
        this.ranking = []; 
        this.salesreps.forEach(salesrep => {
          let salesRep = salesrep;
          let leadScore = parseInt(this.scoreSelect);
          let customerAge = parseInt(this.ageSelect);
          let customerGender = this.genderSelect;
          let customerCountry = this.countrySelect;
          let customerLanguage = this.languageSelect;
          let companySize = parseInt(this.sizeSelect);
          let companyIndustry = this.industrySelect;
          let inputs = [salesRep, leadScore, customerAge, customerGender, customerCountry, customerLanguage, companySize, companyIndustry];
          console.log(inputs); 
          try {
            this.neuralNetwork.classify(inputs, (err, results) => {
                if (err != undefined) {
                  console.log(err);
                } else {
                  this.updateRanking({name: salesrep, success: this.getProperty(results, 'success'), failure: this.getProperty(results, 'failure')});   
                }
            });
          } catch (error) {
            console.log('There was an error during the prediciton.');
            console.log('Please try correcting the input values and try again.'); 
            console.error(error);
          }
        }); 
      },
      getProperty(result, property) {
        return result.find(obj => {
          return obj.label === property
        }).confidence; 
      },
      updateRanking(newEntry) {
        console.log('Update ranking...'); 
        this.ranking.push(newEntry); 
        this.ranking.sort((a, b) => (a.success < b.success) ? 1 : -1); 
        if(this.ranking.length == this.salesreps.length) {
          console.log('Finished ranking: ');
          this.ranking.forEach(rank => {
            console.log(rank.name + ' has a ' + rank.success * 100 + '% chance to succeed.');
          });
          console.log(this.ranking); 
          //setup chart
          var newData = JSON.parse(JSON.stringify(this.series));
          var newOptions = JSON.parse(JSON.stringify(this.chartOptions));
          newData[0].data = []; 
          newOptions.xaxis.categories = []; 
          this.ranking.forEach(rank => {
            newData[0].data.push(rank.success * 100); 
            newOptions.xaxis.categories.push(rank.name); 
          });
          this.series = newData; 
          this.chartOptions = newOptions;
          console.log(this.series); 
          console.log(this.chartOptions); 
          //open dialog
          this.resultDialog = true;
        }
      },
      isInputValid() {
        return (
          this.scoreSelect != null &&
          this.ageSelect != null &&
          this.genderSelect != null &&
          this.countrySelect != null &&
          this.languageSelect != null &&
          this.sizeSelect != null &&
          this.industrySelect != null
        ); 
      }
    },
    mounted() {
      this.setupNeuralNetwork();
    }
  }
</script>
