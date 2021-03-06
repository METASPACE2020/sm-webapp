<template>
  <div>
    <el-row>
      <xic-plot v-if="sameAdductAnnotations"
                :intensityImgs="sameAdductAnnotations.map(a => a.isotopeImages[0])"
                :graphColors="adductColors"
                :acquisitionGeometry="acquisitionGeometry"
                :logIntensity="true"
                :showLogIntSwitch="true">
      </xic-plot>
    </el-row>
    <plot-legend :items="adductLegendItems">
    </plot-legend>
  </div>
</template>

<script>
 import { schemeCategory10 as graphColors } from 'd3';
 import { renderMolFormula } from '../../../util';
 import XicPlot from '../../XicPlot.vue';
 import PlotLegend from '../../PlotLegend.vue';
 import { allAdductsQuery } from '../../../api/annotation';

 export default {
   props: ['annotation', 'database', 'acquisitionGeometry'],
   components: {
     PlotLegend,
     XicPlot
   },
   apollo: {
     sameAdductAnnotations: {
       query: allAdductsQuery,
       variables() {
         return {
           db: this.database,
           datasetId: this.annotation.dataset.id,
           molFormula: this.annotation.sumFormula
         };
       },
       update: data => data.allAnnotations.slice().sort((a, b) => a.mz - b.mz)
     }
   },
   computed: {
     adductColors() {
       if (!this.sameAdductAnnotations) {
         return [];
       }
       // no adducts apart from current annotation
       if (this.sameAdductAnnotations.length == 1) {
         return [graphColors[0]];
       }
       // taking last colors from the palette
       const colors = graphColors.slice(-this.sameAdductAnnotations.length + 1);
       // replacing color of the current annotation with the 1st palette color
       const curAnnIdx = this.sameAdductAnnotations.findIndex(a => a.adduct == this.annotation.adduct);
       colors.splice(curAnnIdx, 0, graphColors[0]);
       return colors;
     },
     adductLegendItems() {
       if (!this.sameAdductAnnotations) {
         return [];
       }
       const colors = this.adductColors;
       return this.sameAdductAnnotations.map((a, idx) => {
         return {
           name: this.showAdduct(a),
           color: colors[idx],
           opacity: 1
         };
       });
     }
   },
   methods: {
     showAdduct(annotation) {
       return `<span>${renderMolFormula(this.annotation.sumFormula, annotation.adduct, this.annotation.dataset.polarity)}</span><br/>
              ${annotation.mz.toFixed(4)}<br/>
              <span style="font-size: smaller">
                MSM score: ${annotation.msmScore.toFixed(3)}<br/>
                Annotated @ ${annotation.fdrLevel * 100}% FDR<br/>
                Max. intensity: ${annotation.isotopeImages[0].maxIntensity.toExponential(2)}
              </div>`;
     }
   }
 }
</script>
