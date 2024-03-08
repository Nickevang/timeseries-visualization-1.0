<template>
  <div class="container">
    <div class="chart-scroll-container" ref="scrollContainer" @mousedown="startScrolling" @mouseup="stopScrolling" @mouseleave="stopScrolling" @mousemove="scrollChart">
      <div class="chart-container" :style="{ width: chartWidth }">
        <CanvasJSChart :options="options" :style="styleOptions" @chart-ref="chartInstance"/>
      </div>
    </div>
    <div class="button-container">
      <div>
        <label for="start">Start Date and Time:</label>
        <input type="datetime-local" id="start" v-model="startDate" @change="updateChart" :min="minDate" :max="maxDate"/>
      </div>
      <div>
        <label for="end">End Date and Time:</label>
        <input type="datetime-local" id="end" v-model="endDate" @change="updateChart" :min="minDate" :max="maxDate"/>
      </div>
    </div>
    <div class="checkbox-container">
      <div v-for="(series, index) in options.data" :key="index">
        <input type="checkbox" v-model="series.visible" @change="updateChart">{{ series.name }}
      </div>
    </div>
  </div>
</template>


<script>
import jsonData from './timeseries.json';

export default {
  data() {
  return {
    chart: null,
    startDate: '',
    endDate: '',
    minDate: '2024-02-01T00:00:00',
    maxDate: '2024-02-12T23:00:00',
    options: {
      theme: "dark2",
      animationEnabled: true,
      exportEnabled: true,
      title: {
        text: "Electricity Prices in Euros",
        fontColor: "white"
      },
      axisY: { 
        title: "Prices (€/MWh)",         
        labelFontColor: "white",
        titleFontColor: "white",
        labelFormatter: (e) => {
          var suffixes = ["", "K", "M", "B"];
          var order = Math.max(Math.floor(Math.log(e.value) / Math.log(1000)), 0);
          if(order > suffixes.length - 1)
            order = suffixes.length - 1;
          var suffix = suffixes[order];
          return (e.value / Math.pow(1000, order)) + suffix;
        }
      },
      axisX: {
        title: "Date",
        intervalType: "hour",
        interval: 1,
        valueFormatString: "DD MMM, YYYY HH:mm",
        labelFontColor: "white",
        titleFontColor: "white"
      },
      legend: {
        cursor: "pointer",
        verticalAlign: "bottom",
        horizontalAlign: "center",
        dockInsidePlotArea: false,
        fontColor: "white"
      },
      data: [{
        type: "spline",
        xValueFormatString: "DD MMM, YYYY HH:mm",
        name: "DE (Germany)",
        showInLegend: true,
        visible: true,
        dataPoints: []
      },
      {
        type: "spline",
        xValueFormatString: "DD MMM, YYYY HH:mm",
        name: "GR (Greece)",
        showInLegend: true,
        visible: true,
        dataPoints: []
      },
      {
        type: "spline",
        xValueFormatString: "DD MMM, YYYY HH:mm",
        name: "FR (France)",
        showInLegend: true,
        visible: true,
        dataPoints: []
      }]
    },
    styleOptions: {
      width: "100%",
      height: "360px"
    },
    chartWidth: '800px', // Initial chart width
    scrolling: true,
    startX: 0,
    startScrollLeft: 0
  }
},

  mounted() {
    // Parse and render chart with data from local JSON file
    this.parseDataAndRenderChart();
  },
  methods: {
    parseDataAndRenderChart() {
      // Access data from imported JSON file
      for (let i = 0; i < jsonData.length; i++) {
        this.options.data[0].dataPoints.push({ 
          x: new Date(jsonData[i]["DateTime"]), 
          y: parseFloat(jsonData[i]["ENTSOE_DE_DAM_Price"]) // Convert to float
        });
        
        this.options.data[1].dataPoints.push({ 
          x: new Date(jsonData[i]["DateTime"]), 
          y: parseFloat(jsonData[i]["ENTSOE_GR_DAM_Price"]) // Convert to float
        });

        this.options.data[2].dataPoints.push({ 
          x: new Date(jsonData[i]["DateTime"]), 
          y: parseFloat(jsonData[i]["ENTSOE_FR_DAM_Price"]) // Convert to float
        });
      }
      // Render the chart
      this.chart.render()
    },
    chartInstance(chart) {
      // Store reference to the chart instance
      this.chart = chart;
    },
    updateChart() {
      // Filter data based on selected start and end dates
      const filteredData = jsonData.filter(entry => {
        const entryDate = new Date(entry["DateTime"]);
        return entryDate >= new Date(this.startDate) && entryDate <= new Date(this.endDate);
      });

      // Clear existing data
      this.options.data.forEach(series => {
        series.dataPoints = [];
      });

      // Update chart data with filtered data
      filteredData.forEach(entry => {
        this.options.data[0].dataPoints.push({ 
          x: new Date(entry["DateTime"]), 
          y: parseFloat(entry["ENTSOE_DE_DAM_Price"])
        });
        this.options.data[1].dataPoints.push({ 
          x: new Date(entry["DateTime"]), 
          y: parseFloat(entry["ENTSOE_GR_DAM_Price"])
        });
        this.options.data[2].dataPoints.push({ 
          x: new Date(entry["DateTime"]), 
          y: parseFloat(entry["ENTSOE_FR_DAM_Price"])
        });
      });

      // Calculate chart width based on the number of data points
      const numDataPoints = filteredData.length;
      this.chartWidth = `${Math.max(800, numDataPoints * 10)}px`;

      // Re-render the chart
      this.chart.render();
    },
    startScrolling(event) {
      this.scrolling = true;
      this.startX = event.clientX;
      this.startScrollLeft = this.$refs.scrollContainer.scrollLeft;
    },
    stopScrolling() {
      this.scrolling = false;
    },
    scrollChart(event) {
      if (this.scrolling) {
        const deltaX = event.clientX - this.startX;
        this.$refs.scrollContainer.scrollLeft = this.startScrollLeft - deltaX;
      }
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #222;
  padding: 20px;
}

.chart-scroll-container {
  overflow-x: auto;
  margin-bottom: 20px;
}

.chart-container {
  width: 100%;
  max-width: 100%;
}

.button-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}

.button-container div {
  margin: 0 10px;
}

.button-container label {
  color: white;
}

button {
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #45a049;
}

.checkbox-container  {
  color: white;
}
</style>
