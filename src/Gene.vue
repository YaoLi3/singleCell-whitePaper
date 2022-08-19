<template>
  <div id='app'>
    <title>Cluster Table</title>

    <div>
        <!-- searchable header -->
        <p class="inline_item" > Species:</p>
        <el-select class="inline_item" v-model='currentSpecies' filterable placeholder="">
          <el-option v-for="item in Ssamples" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
        <p class="inline_item" > Celltype:</p>
        <el-select class="inline_item" v-model='currentCellType' filterable placeholder="all cell types" @change='selectCellType'>
          <el-option v-for="item in samples" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>

        <p class="inline_item" > AUC cutoff:</p>
        <el-input  class="inline_item" v-model='aucCutoff' @input.native="searchGene" style='width:150px;' placeholder="0" type='number' max=1, min=0></el-input>
        <p class="inline_item" > Search Gene:</p>
        <el-input  class="inline_item" v-model='currentGene' style='width:150px;' placeholder="contig id"></el-input>
        <!-- searchable header end -->

        <!-- cluster table content -->
        <el-table ref="clusterTable" 
        :show-header='true' class="table" 
        :data="tableData.slice((currentPage-1)*pageSize,currentPage*pageSize)"
        :highlight-current-row='true'
        stripe
        @row-click='handleRow'>
            <el-table-column prop='Contig' label='Contig'></el-table-column>
            <el-table-column prop='Best-blast hit' label='Best-blast hit'></el-table-column>
            <el-table-column prop='Accession' label='Accession'></el-table-column>
            <el-table-column prop='e-value' label='e-value'></el-table-column>
            <el-table-column prop='Organism' label='Organism'></el-table-column>
            <el-table-column prop='log fold enrichment' label='log fold enrichment'></el-table-column>
            <el-table-column prop='cluster enriched in' label='cluster enriched in'></el-table-column>
            <el-table-column prop='Adjusted p-value' label='Adjusted p-value'></el-table-column>
            <el-table-column prop='AUC' label='AUC'></el-table-column>
            <el-table-column prop='Power' label='Power'></el-table-column>
            <el-table-column prop='Associated cell  type class' label='Associated cell type class'></el-table-column>
        </el-table>
        <el-pagination layout="total,sizes, prev, jumper, next"
        :total="this.tableData.length"
        :current-page="currentPage"
        @current-change="handleCurrentChange" @size-change="handleSizeChange"
        :page-sizes="[10,15,20]" :page-size="pageSize" :current-page.sync="currentPage">
        </el-pagination>
        <!-- cluster table content -->
    </div>


  </div>
</template>

<script>
    import $ from 'jquery';

    // datasets
    var GENE_DATA_URL = "http://49.235.68.146/gene_data/All_Clusters.json"


    export default {
        data(){
            return {
                // data examples :
                Ssamples : [ { index:1, value:"Planarian",},
                            { index:2, value:"Zebrafish",},
                            { index:3, value:"Salamander",},
                            { index:4, value:"Shark",},
                            { index:5, value:"Whale",}, ],
                samples : [ { index:1, value:"Neoblast",},
                            { index:2, value:"Neural",},
                            { index:3, value:"Cathepsin+ cell",},
                            { index:4, value:"Epidermal",},
                            { index:5, value:"Intestine",},
                            { index:6, value:"Muscle",},
                            { index:7, value:"Phyarnx",},
                            { index:8, value:"Protonephridia",},
                            { index:9, value:"Parapharyngeal",},
                            { index:10, value:"-",}, 
                            { index:11, value:"All",},],
                currentSpecies:'Planarian',
                aucCutoff: 0,
                currentGene: null,
                currentCellType: null,
                tableData: [],
                allTableData: [],
                pageSize:15,
                currentPage:1,
            }; // end of data return
        },
        methods: {
            selectCellType(item){
                //this.currentCellType = item;
                if (this.currentCellType == 'All'){
                    this.tableData = this.allTableData;
                    return
                }
                // change table data
                var new_tableData = [];
                var arrayLength = this.allTableData.length;
                for (var i = 0; i < arrayLength; i++) {
                    if (this.allTableData[i]['Associated cell  type class'] == this.currentCellType){
                        new_tableData.push(this.allTableData[i]);
                    }
                }
                this.tableData = new_tableData;
            },
            selectSample(item){
                console.log(item);
                if(item == this.currentSpecies)
                    return
                this.currentSpecies = item;
                //this.tableData = getSpeciesData(this.currentSpecies);
            },
            searchGene(item){
                console.log(currentGene);
                console.log(item);            
            },
            handleSizeChange (size) {
                this.pageSize = size;
            },
            handleCurrentChange (currentpage){
                this.currentPage = currentpage;
            },
            handleRow(row,event,column){
                this.currentGene = row.Contig;
                console.log(this.currentGene);
            }
        },
        beforeMount(){
            var self = this;
            $.getJSON(GENE_DATA_URL, function(_data){
                self.tableData = _data;
                self.allTableData = _data;
            });
        },
    };
</script>

<style>
.inline_item {
    display: inline-block;
    margin-left: 20px;
    vertical-align: middle;
}
</style>
