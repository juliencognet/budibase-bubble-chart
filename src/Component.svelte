<script>
  import { getContext,afterUpdate } from "svelte"
  import { chart } from "svelte-apexcharts";
  import { merge } from 'lodash';
  
  export let dataProvider //
  export let serieField //
  export let xAxisField // 
  export let yAxisField //
  export let bubbleSizeField // 
  export let xAxisType //
  export let chartHeight //
  export let chartWidth //
  export let chartTitle //
  export let minXAxis //
  export let maxXAxis //
  export let xAxisTickAmount //
  export let minYAxis //
  export let maxYAxis //
  export let yAxisTitle //
  export let yAxisTickAmount //
  export let displayValues //
  export let bubbleOpacity //
  export let colorPalette //
  export let advancedChartOptions //

  const loading = getContext("loading")
  let canBeDisplayed = true;
  let errorMessages = [];

  // ----------------------------------------------------------------
  // Check configuration of chart
  if (dataProvider==null){
    errorMessages.push("Data provider must be defined.");
  }
  const xAxisAuthorizedFieldTypesAndMapping = [
    {"fieldType":"datetime","mappedChartType":"datetime"}
   ,{"fieldType":"number","mappedChartType":"numeric"}
   ,{"fieldType":"string","mappedChartType":"category"}
   ,{"fieldType":"barcodeqr","mappedChartType":"category"}
   ,{"fieldType":"options","mappedChartType":"category"}
   ]
  const errorIfNotInAuthorizedTypes = "must be either a date, a number, a string, options or a barcode"
  if (serieField!=null){
    if (xAxisAuthorizedFieldTypesAndMapping.find(elt=>elt.fieldType==dataProvider.schema[serieField].type)===undefined){
      errorMessages.push("Serie field "+errorIfNotInAuthorizedTypes);
    }
  }
  if (xAxisField==null) {
    errorMessages.push("xAxis Field must be defined");
  } else if (xAxisAuthorizedFieldTypesAndMapping.find(elt=>elt.fieldType==dataProvider.schema[xAxisField].type)===undefined){
    errorMessages.push("xAxis Field "+errorIfNotInAuthorizedTypes);
  } else {
    xAxisType = xAxisAuthorizedFieldTypesAndMapping.find(elt=>elt.fieldType==dataProvider.schema[xAxisField].type).mappedChartType;
  }
  if (yAxisField==null) {
    errorMessages.push("yAxis Field must be defined");
  } else if (dataProvider.schema[yAxisField].type != 'number') {
    errorMessages.push("yAxis Field must be a number field")
  } 
  if (bubbleSizeField==null){
    errorMessages.push("Bubble size field must be defined");
  } else if (dataProvider.schema[bubbleSizeField].type != 'number') {
    errorMessages.push("Bubble size field must be a number field");
  }
  if (errorMessages.length>0){
    canBeDisplayed=false;
  }

  // ----------------------------------------------------------------
  // Manage all options of the chart
  var options = {};
  options.chart = { height: chartHeight, type: 'bubble', toolbar: {show: false}}
  if (chartWidth!=null){
    options.chart.width = chartWidth;
  }
  if (chartTitle!=null){
    options.title = { text: chartTitle }
  }
  options.xaxis = { type: xAxisType }
  if (minXAxis!=null){
    options.xaxis.min = minXAxis;
  }
  if (maxXAxis!=null){
    options.xaxis.max = maxXAxis;
  }
  if (xAxisTickAmount!=null){
    options.xaxis.tickAmount = xAxisTickAmount;
  }
  options.yaxis = {};
  if (yAxisTitle!=null){
    options.yaxis.title = { text: yAxisTitle };
  }
  if (minYAxis!=null){
    options.yaxis.min = minYAxis;
  }
  if (maxYAxis!=null){
    options.yaxis.max = maxYAxis;
  }
  if (yAxisTickAmount!=null){
    options.yaxis.tickAmount = yAxisTickAmount;
  }
  options.dataLabels = { enabled: displayValues };
  options.fill = { opacity : bubbleOpacity/100 };
  options.theme = {palette : colorPalette };

  // Merge advanced chart options with options computed just before
  if (advancedChartOptions!=null){
    options = merge(options, advancedChartOptions)
  }

  // ----------------------------------------------------------------
  // Manage data of the chart
  options.series = [];
  if (!serieField){
    options.series.push({"name":"DEFAULT",data: []});
  }

  // Handle data when rows retrieved
  function handleData(rows){
    let index=0;
    options.series=[];
    if (rows.length>0){
      for (const element of rows){
        try{
          // Manage serie if defined (else will be DEFAULT value)
          if (serieField == null){
            index=0;
          } else {
            let categoryName = element[serieField];
            index = options.series.findIndex( el => el.name == categoryName);
            if (index==-1){
              index = options.series.push({"name": categoryName, data: []}) -1 ;
            }
          }
          // If no value defined for x axis, but that axis if category type, then we can do something, else value will be ignored
          if (element[xAxisField]!==undefined || options.xaxis.type=="category") {
            let xValue = (element[xAxisField]===undefined) ? "<undefined>" : element[xAxisField]
            if (xValue != null && element[yAxisField] && element[bubbleSizeField] != null){
              options.series[index].data.push(
                {x: xValue
                ,y:element[yAxisField]
                ,z:element[bubbleSizeField]});
            }
          }
        } catch (Exception){
          console.log("Could not add element to the bubble chart")
        }
      };
    }
  }
  // Code executed at startup of component
  $: if (dataProvider?.rows){
      handleData(dataProvider?.rows);
  } 
  // Code executed on update of binded values
  afterUpdate(() => {
    if (dataProvider?.rows){
      handleData(dataProvider?.rows);
    } 
	});
</script>

{#if canBeDisplayed}
  <div use:chart={options} />
{:else}
<!-- Display error message of bad configuration -->
<div class="spectrum-InLineAlert spectrum-InLineAlert--notice">
  <div class="spectrum-InLineAlert-header">
    Bubble chart configuration error
    <svg class="spectrum-Icon spectrum-Icon--sizeM spectrum-InLineAlert-icon" focusable="false" aria-hidden="true">
      <use xlink:href="#spectrum-icon-18-Alert" />
    </svg>
  </div>
  <div class="spectrum-InLineAlert-content">
    <ul>
      {#each errorMessages as message}
        <li>{message}</li>
      {/each}
    </ul>
  </div>
</div>
{/if}