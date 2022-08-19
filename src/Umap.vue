<template>
  <div>
      <div style="margin-left:0%;background-color: rgb(238, 241, 246); border: 3px solid #eee;" >
          <!-- select gene start -->
          <p class="inline_item" > Please select one species:</p>
          <el-select class="inline_item" v-model="curr_selected_sample" filterable placeholder="" @change="selectSample" >
            <el-option v-for="item in samples" :key="item.value"
               :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          <!-- select gene start -->
          <p class="inline_item" > Please type the gene name:</p>
          <el-input  class="inline_item" style='width:250px;' v-model="input_gene" placeholder=""></el-input>
         <el-button class='inline_item' @click.native="submit_gene">Submit</el-button>
          <!-- select gene start -->
          <p class="inline_item" > Or try examples:</p>
          <el-select class="inline_item" v-model="selected_example" filterable placeholder="" @change="selectGene" >
            <el-option v-for="item in genes" :key="item.value"
               :label="item.value" :value="item.value">
            </el-option>
          </el-select>
      </div>
      <div>
          <div class="inline_item">
              <p>The celltype in Umap2D space:</p>
              <v-chart ref="my_cell_echarts"  :option="cell_option" style="width:500px;height:500px;" />
          </div>
          <div class="inline_item">
              <p>The gene in Umap2D space:</p>
              <v-chart ref="my_gene_echarts"  :option="gene_option" style="width:500px;height:500px;" />
          </div>
      </div>
  </div>
</template>

<script>
  import $ from 'jquery';
  import * as echarts from 'echarts';
  import VChart from "vue-echarts";
  var CT_URL='http://49.235.68.146/cluster_data/Planarian.CellType.json';
  var GENE_URL='http://49.235.68.146/gene_data/single_gene/'
  export default {
    name : "Umap2D",
    components: {
        VChart
    },
    data(){
      return {
        // data buffering
        input_gene : null,
        selected_example : null,
        curr_cell_data : null,
        curr_gene_data : null,
        // selection tags :
        curr_selected_sample : 'Planarian',
        curr_selected_gene : null,
        // data examples :
        samples : [ { index:1, value:"Planarian",},
                    { index:2, value:"Zebrafish",},
                    { index:3, value:"Salamander",},
                    { index:4, value:"Shark",},
                    { index:5, value:"Whale",}, ],

        genes : [   { index:1, value:"dd_Smed_v4_1000_0_1",},
                    { index:2, value:"dd_Smed_v4_13053_0_1",},
                    { index:3, value:"dd_Smed_v4_13056_0_1",},
                    { index:4, value:"dd_Smed_v4_13061_0_1",},
                    { index:5, value:"dd_Smed_v4_13086_0_1",},
                    { index:6, value:"dd_Smed_v4_13087_0_1",},
                ],
        // echarts options
        gene_option : {backgroundColor:'#FFFFFF',},
        cell_option : {backgroundColor:'#FFFFFF',},
       };
     },
     methods: {
       /*********************functions for datas start**********************/
       clean_gene_buffer(){
         this.curr_selected_gene == null;
         this.curr_gene_data = null;
         this.$refs.my_gene_echarts.setOption(this.get_gene_option(),true);
       },
       clean_cell_buffer(){
         this.curr_selected_gene == null;
         this.curr_cell_data = null;
         this.$refs.my_cell_echarts.setOption(this.get_cell_option(),true);
       },
       clean_buffer(){
         this.clean_gene_buffer();
         this.clean_cell_buffer();
       },
       loading_gene_data(){
          var self = this;
          var used_url = GENE_URL+this.curr_selected_gene+'.gene_expr.json';
          console.log('start loading gene')
          $.getJSON(used_url,function(_data) {
             console.log('loading gene done')
             self.curr_gene_data = _data 
             self.gene_option = self.get_gene_option();
          });
          return;
       },
       loading_cell_data(){
           var self = this;
           var used_url = CT_URL;
           console.log('start loading cell')
           $.getJSON(used_url,function(_data) {
             console.log('loading cell done')
             self.curr_cell_data = _data 
             self.cell_option = self.get_cell_option();
             self.gene_option = self.get_gene_option();
           });
       },
       /*********************functions for datas end**********************/

       /*********************functions for user selection start**********************/
       submit_gene(){
          this.selected_example = null;
          this.selectGene(this.input_gene);
       },
       selectGene(item){
         console.log(item);
         this.clean_gene_buffer()
         this.curr_selected_gene = item;
         this.loading_gene_data()
       },

       selectSample(item){
         console.log(item);
         if(item == this.curr_selected_sample)
             return
         this.clean_buffer();
         this.curr_selected_sample = item;
         this.loading_cell_data();
       },
       /*********************functions for user selection end  **********************/

       /*********************functions for echarts option start**********************/
       placeholder_option(msg){
         return  {
           backgroundColor:'#FFFFFF',
           title : {
               text : msg,
               left: "center",
               top: "center",
               textStyle: {
                  color: '#000000'
               },
           }
         };
       },
       get_cell_option(){
          if( this.curr_cell_data == null ) {
              return this.placeholder_option('Loading data ... ');
          }
          var series_list = [];
          for (const [celltype, coords] of Object.entries(this.curr_cell_data)) {
              var one_ct = {
                 name: celltype,
                 type: 'scatter',
                 data: coords,
                 dimensions: ['x', 'y'],
                 symbolSize: 3,
                 itemStyle: {
                   opacity: 0.6
                 },
                 large: true
              };
              series_list.push(one_ct)
          };
          var option = {
              backgroundColor:'#FFFFFF',
              title : {
                   text : '',
                   left: "center",
                   top: "center",
                   textStyle: {
                      color: '#000000'
                   },
              },
              legend: {
                  // orient: 'vertical',
              },
              xAxis: [{}],
              yAxis: [{}],
              toolbox: {
                show: true,
                feature: {
                    saveAsImage: {},
                }
              },
              tooltip: {},
              series: series_list
          };
          return option;
       },
       get_gene_option(){
         if( this.curr_cell_data == null ) {
             return this.placeholder_option('Waiting species data...');
         }
         var series_list = [];
         var sid = 0;
         for (const [celltype, coords] of Object.entries(this.curr_cell_data)) {
             var one_ct = {
                type: 'scatter',
                data: coords,
                dimensions: ['x', 'y'],
                symbolSize: 2,
                color:'#0f0f0f',
                itemStyle: {
                  opacity: 0.1
                },
                large: true
             };
             series_list.push(one_ct);
             sid = sid + 1;
         };
         var msg = '';
         if (this.curr_selected_gene == null)
             msg = 'Please select a specfic gene to display ...';
         else if (this.curr_gene_data == null ) 
             msg = 'Loading data ...';
         else {
             var data = [];
             var one_gene = {
                name : this.curr_selected_gene,
                type: 'scatter',
                symbolSize: 5,
                data: this.curr_gene_data,
                itemStyle: {
                  opacity: 0.8
                },
             };
             series_list.push(one_gene)
         }
         var option = {
              backgroundColor:'#FFFFFF',
              title : {
                   text : msg,
                   left: "center",
                   top: "center",
                   textStyle: {
                      color: '#000000'
                   },
              },
              xAxis: [
                 {
                    type: 'value'
                 }
              ],
              yAxis: [
                 {
                  type: 'value'
                 }
              ],
              toolbox: {
                show: true,
                feature: {
                    saveAsImage: {},
                }
              },
              tooltip: {},
              series: series_list
         };
         if( this.curr_cell_data != null &&  this.curr_gene_data != null )
         {
            option.visualMap= {
                  min: 0,
                  max: 5,
                  dimension: 2,
                  orient: 'vertical',
                  seriesIndex: sid  ,
                  right: 5,
                  top: 'center',
                  calculable: true,
                  inRange: {
                    color: ['blue', 'white','red']
                  }
                };
         }
         return option;
       },
       /*********************functions for echarts option end**********************/
     },
     mounted(){
         this.gene_option = this.get_gene_option();
         this.cell_option  = this.get_cell_option();
         if( this.curr_selected_sample != null )
             this.loading_cell_data()
     },
  };
</script>

<style>
.inline_item_tight {
  display: inline-block;
  margin-left: 3px;
  vertical-align: middle;
}
.inline_item {
  display: inline-block;
  margin-left: 20px;
  vertical-align: middle;
}
</style>
