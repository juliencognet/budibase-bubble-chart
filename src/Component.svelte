<script>
  import { getContext, setContext } from "svelte"
  import { chart } from "svelte-apexcharts";
  import { merge } from 'lodash';
  const loading = getContext("loading")

  export let dataProvider
  export let serieField //
  export let xAxisField // 
  export let yAxisField //
  export let bubbleSizeField // 
  export let xAxisType //
  export let chartHeight //
  export let chartTitle //
  export let minXAxis //
  export let maxXAxis //
  export let xAxisTitle //
  export let xAxisTickAmount //
  export let minYAxis //
  export let maxYAxis //
  export let yAxisTitle //
  export let yAxisTickAmount //
  export let displayValues //
  export let bubbleOpacity //
  export let colorPalette //
  export let advancedChartOptions //

  // Manage all options of the chart
  var options = {};
  options.chart = { height: chartHeight, type: 'bubble', toolbar: {show: false}}
  if (chartTitle){
    options.title = { text: chartTitle }
  }
  options.xaxis = { type: xAxisType };
  if (xAxisTitle){
    options.xaxis.title = { text: xAxisTitle, offsetY: +120 };
  }
  if (minXAxis){
    options.xaxis.min = minXAxis;
  }
  if (maxXAxis){
    options.xaxis.max = maxXAxis;
  }
  if (xAxisTickAmount){
    options.xaxis.tickAmount = xAxisTickAmount;
  }
  options.yaxis = {};
  if (yAxisTitle){
    options.yaxis.title = { text: yAxisTitle };
  }
  if (minYAxis){
    options.yaxis.min = minYAxis;
  }
  if (maxYAxis){
    options.yaxis.max = maxYAxis;
  }
  if (yAxisTickAmount){
    options.yaxis.tickAmount = yAxisTickAmount;
  }
  options.dataLabels = { enabled: displayValues };
  options.fill = { opacity : bubbleOpacity/100 };
  options.theme = {palette : colorPalette };
  // Merge options
  if (advancedChartOptions!=null){
    options = merge(options, advancedChartOptions)
  }
  // Manage data of the chart
  options.series = [];
  if (!serieField){
    options.series.push({"name":"DEFAULT",data: []});
  }
  $: { // asynchronous portion of code which can be executed several times
       // until data are loaded
    let rows = $loading
    ? new Array(dataProvider?.limit > 20 ? 20 : dataProvider?.limit).fill({})
    : dataProvider?.rows
    let index=0;
    for (const element of rows){
        if (serieField == null){
          index=0;
        } else {
          let categoryName = element[serieField];
          index = options.series.findIndex( el => el.name == categoryName);
          if (index==-1){
            index = options.series.push({"name": categoryName, data: []}) -1 ;
          }
        }
        options.series[index].data.push({x:element[xAxisField],y:element[yAxisField],z:element[bubbleSizeField]});
    };
  }
</script>

<div use:chart={options} />
