<style>
  .slider {
    -webkit-appearance: none;
    width: 100%;
    height: 15px;
    border-radius: 5px;
    background: #d3d3d3;
    outline: none;
    opacity: 0.7;
    -webkit-transition: .2s;
    transition: opacity .2s;
  }

  .slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 25px;
    height: 25px;
    border-radius: 50%;
    background: #4CAF50;
    cursor: pointer;
  }

  .slider::-moz-range-thumb {
    width: 25px;
    height: 25px;
    border-radius: 50%;
    background: #4CAF50;
    cursor: pointer;
  }
</style>
<script>
  import { onMount } from 'svelte'
  import AutoComplete from './AutoComplete.svelte'

  const Chart = require('chart.js')
  let status = 'Confirmed'
  const stateCodes = {
    AP: 'Andhra Pradesh',
    AR: 'Arunachal Pradesh',
    AS: 'Assam',
    BR: 'Bihar',
    CT: 'Chhattisgarh',
    GA: 'Goa',
    GJ: 'Gujarat',
    HR: 'Haryana',
    HP: 'Himachal Pradesh',
    JH: 'Jharkhand',
    KA: 'Karnataka',
    KL: 'Kerala',
    MP: 'Madhya Pradesh',
    MH: 'Maharashtra',
    MN: 'Manipur',
    ML: 'Meghalaya',
    MZ: 'Mizoram',
    NL: 'Nagaland',
    OR: 'Odisha',
    PB: 'Punjab',
    RJ: 'Rajasthan',
    SK: 'Sikkim',
    TN: 'Tamil Nadu',
    TG: 'Telangana',
    TR: 'Tripura',
    UT: 'Uttarakhand',
    UP: 'Uttar Pradesh',
    WB: 'West Bengal',
    AN: 'Andaman and Nicobar Islands',
    CH: 'Chandigarh',
    DN: 'Dadra and Nagar Haveli',
    DD: 'Daman and Diu',
    DL: 'Delhi',
    JK: 'Jammu and Kashmir',
    LA: 'Ladakh',
    LD: 'Lakshadweep',
    PY: 'Puducherry',
  }
  let days = 30
  let total_days
  let cumulative = true

  let whitelisted_states = ['mh', 'gj','dl', 'tn', 'mp', 'rj']
  let state_daily
  let remaining_states = []
  let colors = {}
  let selectedColorObject
  let uinque_colors = ['#E52B50', '#FFBF00', '#9966CC', '#FBCEB1', '#7FFFD4', '#007FFF', '#89CFF0', '#F5F5DC', '#000000', '#0000FF', '#0095B6', '#8A2BE2', '#DE5D83', '#CD7F32', '#964B00', '#800020', '#702963', '#960018', '#DE3163', '#007BA7', '#F7E7CE', '#7FFF00', '#7B3F00', '#0047AB', '#6F4E37', '#B87333', '#FF7F50', '#DC143C', '#00FFFF', '#EDC9Af', '#7DF9FF', '#50C878', '#00FF3F', '#FFD700']

  let c = 0
  for (let s in stateCodes) {
    colors[s.toLowerCase()] = uinque_colors[c]
    c++
  }

  // colors

  function update_remaing_states () {
    remaining_states = []
    for (let s in stateCodes) {
      if (!whitelisted_states.includes(s.toLowerCase())) {
        remaining_states.push({
          abbr: s.toLowerCase(),
          name: stateCodes[s]
        })
      }
    }
    remaining_states = remaining_states
  }

  update_remaing_states()

  let addNew

  function update_graph (res, whitelisted_states, status, days, cumulative) {
    let datasets_temp = {
      'an': '0',
      'ap': '1',
      'ar': '0',
      'as': '0',
      'br': '0',
      'ch': '0',
      'ct': '0',
      'dd': '0',
      'dl': '7',
      'dn': '0',
      'ga': '0',
      'gj': '0',
      'hp': '0',
      'hr': '14',
      'jh': '0',
      'jk': '0',
      'ka': '6',
      'kl': '19',
      'la': '0',
      'ld': '0',
      'mh': '14',
      'ml': '0',
      'mn': '0',
      'mp': '0',
      'mz': '0',
      'nl': '0',
      'or': '0',
      'pb': '1',
      'py': '0',
      'rj': '3',
      'sk': '0',
      'tg': '1',
      'tn': '1',
      'tr': '0',
      'tt': '79',
      'up': '12',
      'ut': '0',
      'wb': '0'
    }
    let datasets = []

    for (let s in datasets_temp) {
      if (whitelisted_states.includes(s)) {
        datasets.push({
          label: s,
          fill: false,
          borderColor: colors[s],
          data: [0],
        })
      }
    }
    let labels = ['']
    let cases = res.data.states_daily.filter((d) => d.status === status)
    total_days = cases.length
    cases = cases.splice(total_days - days, total_days)
    for (let day in cases) {
      labels.push(cases[day].date)
      for (let states in cases[day]) {
        if (states !== 'date' && states !== 'status') {
          if (whitelisted_states.includes(states)) {
            for (let s in datasets) {
              if (datasets[s].label === states) {
                let element = parseInt(cases[day][states])
                if (cumulative) {
                  element += datasets[s].data[datasets[s].data.length - 1]
                }
                datasets[s].data.push(element)
              }
            }
          }
        }
      }
    }

    for (let d in datasets) {
      datasets[d].label = stateCodes[datasets[d].label.toUpperCase()]
    }

    document.getElementById('stateChartWrapper').innerHTML=""
    let ctx_element = document.createElement("canvas");
    ctx_element.setAttribute('class','w-100')
    ctx_element.height = "600"
    document.getElementById('stateChartWrapper').appendChild(ctx_element)
    let ctx = ctx_element.getContext('2d')
    let chart = new Chart(ctx, {
      type: 'line',
      data: {
        datasets: datasets,
        labels: labels
      },
      options: {
        title: {
          display: true,
          text: `Statewise cases of COVID-19 in India of last ${days}`,
          fontSize: 25
        },
        scales: {
          yAxes: [{
            scaleLabel: {
              display: true,
              labelString: 'No. of cases'
            }
          }],
          xAxes: [{   ticks: {
              autoSkip: true,
              maxTicksLimit: 10
            },
            distribution: 'series'
          }]
        }

      }
    })
  }

  onMount(async () => {
    if (!state_daily) {
      state_daily = await window.api('https://api.covid19india.org/states_daily.json')
    }
    update_graph(state_daily, whitelisted_states, status, days, cumulative)
  })

  function add_state () {
    if (addNew) {
      if (!whitelisted_states.includes(addNew)) {
        whitelisted_states.push(addNew)
        update_remaing_states()
        update_graph(state_daily, whitelisted_states, status, days, cumulative)
      }
    }
  }

  function update () {
      update_graph(state_daily, whitelisted_states, status, days, cumulative)
  }

  $: if (status) {
    if(state_daily && days){
      update_graph(state_daily, whitelisted_states, status, days, cumulative)
    }
  }


</script>
<div class="row mt-5">
  <div id="stateChartWrapper" class="col-md-8"></div>
  <div class="col-md-4">
    <div class="text-center border border-light p-5">
      <p class="h4 mb-4">Change Graph parameters</p>
      <AutoComplete
        items={remaining_states}
        bind:selectedItem={selectedColorObject}
        bind:value={addNew}
        valueFieldName="abbr"
        labelFieldName="name"
        onChange={add_state}
        placeholder="Add New states here"
      />

      <p class="h5 mt-4">Select graph of</p>
      <div class="custom-control custom-radio">
        <input type="radio" class="custom-control-input" id="confirmedId"
               value="Confirmed" bind:group={status} checked>
        <label class="custom-control-label" for="confirmedId">Confirmed</label>
      </div>
      <div class="custom-control custom-radio">
        <input type="radio" class="custom-control-input" id="RecoveredId"
               value="Recovered" bind:group={status}>
        <label class="custom-control-label" for="RecoveredId">Recovered</label>
      </div>
      <div class="custom-control custom-radio">
        <input type="radio" class="custom-control-input" id="DeceasedId"
               value="Deceased" bind:group={status}>
        <label class="custom-control-label" for="DeceasedId">Death</label>
      </div>
      <div class="row mt-3">
        <label class="form-control-label col-md-2" for="DaysId">Days</label>
        <input type="range" id="DaysId" class="slider col-md-8" bind:value={days} max="{total_days}"
               min="1">
        <p class="col-md-2 h5">{days}</p>
      </div>
      <div class="custom-control custom-switch row">
        <input type="checkbox" class="custom-control-input" bind:checked={cumulative} on:change={update}
               id="customSwitch1">
        <label class="custom-control-label" for="customSwitch1">Add previous day values(cumulative)</label>
      </div>
    </div>
  </div>
</div>
