<template>

  <a class="button mb-6 is-small" @click="genPermalink">
    Permalink
  </a>

  <a v-show="false" ref="permalinkHref" class="button mr-2 _is-small" target="_blank" rel="noopener noreferrer" :href="permalink"></a>

  <section class="section">


    <label class="label">Qual o valor de pH estimado da amostra?</label>
    <div class="field has-addons">
      <div class="control">
        <input class="input" type="number" placeholder="" v-model="state.valorEstimadoAmostra" style="width: 120px;">
      </div>
      <div class="control">
        <a class="button is-static">
          pH
        </a>
      </div>
    </div>



    <hr>
    <h4 class="title is-4">Quais os dados do certificado dos MRCs a serem utilizados?</h4>

    <div v-for="(mrc, mrc_i) in state.mrcs" :key="mrc_i">
      <div class="field is-grouped is-grouped-multiline">
        
        <div class="field has-addons mr-2 mb-4">
          <div class="control">
            <a class="button is-static">
              Valor
            </a>
          </div>
          <div class="control">
            <input class="input" type="number" placeholder="" v-model="state.mrcs[mrc_i].valor" style="width: 120px;">
          </div>
          <div class="control">
            <a class="button is-static">
              pH
            </a>
          </div>
        </div>

        <div class="field has-addons mr-2">
          <div class="control">
            <a class="button is-static">
              U
            </a>
          </div>
          <div class="control">
            <input class="input" type="number" placeholder="" v-model="state.mrcs[mrc_i].U" style="width: 120px;">
          </div>
          <div class="control">
            <a class="button is-static">
              pH
            </a>
          </div>
        </div>

      </div>


    </div>

    <hr>
    <h4 class="title is-4">Configure seu instrumento para realizar medições em tensão elétrica (mV)</h4>

    <label class="checkbox">
      <input type="checkbox">
      Configurado
    </label>




    <div v-for="(mrc, mrc_i) in state.mrcs" :key="mrc_i">
      <hr>
      <h4 class="title is-4">Insira o eletrodo no MRC de {{mrc.valor}} pH e colete 4 leituras</h4>
        
      <div class="field has-addons mr-2" v-for="(leitura, leitura_i) in mrc.leituras" :key="leitura_i">
        <div class="control">
          <input class="input" type="number" :placeholder="'leitura ' + (leitura_i+1)" v-model="mrc.leituras[leitura_i]" style="width: 120px;">
        </div>
        <div class="control">
          <a class="button is-static">
            mV
          </a>
        </div>
      </div>

      {{ mrcsStat[mrc_i] }}
    </div>


    <div v-for="(amostra, amostra_i) in state.amostras" :key="amostra_i">
      <hr>
      <h4 class="title is-4">Insira o eletrodo na amostra e colete 4 leituras</h4>

        
        <div class="field has-addons mr-2" v-for="(leitura, leitura_i) in amostra.leituras" :key="leitura_i">
          <div class="control">
            <input class="input" type="number" :placeholder="'leitura ' + (leitura_i+1)" v-model="amostra.leituras[leitura_i]" style="width: 120px;">
          </div>
          <div class="control">
            <a class="button is-static">
              mV
            </a>
          </div>
        </div>
    </div>
    {{amostrasStat}}

    <hr>

    <h4 class="title is-4">Resultados</h4>

    <p>Valor medido de pH: {{}}</p>
    K'={{ kLinha }}

    <br>

    <table class="table is-striped">
      <thead>
        <tr>
          <th>Descrição</th>
          <th>u</th>
          <th>Divisor</th>
          <th>Coef Sens</th>
          <th>u_pad</th>
          <th>%</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in calculosKLinha" :key="row[0]">
          <td v-for="(col, col_i) in row" :key="col_i">{{col}}</td>
        </tr>
      </tbody>
    </table>
    
  </section>

</template>

<script>


export default {
  // components: {
  //   RuleNode
  // },
  name: 'HelloWorld',
  props: {
    
  },
  mounted () {
    let urlString = (window.location.href.split('/#/').slice(1).join('/#/'))

    if (urlString.length) {
      let notebookStr = (decodeURIComponent(urlString))
      this.state = JSON.parse(notebookStr)
    }
  },
  data () {
    return {
      permalink: '',
      state: {
        valorEstimadoAmostra: 4,
        mrcs: new Array(2)
          .fill()
          .map((_, i) => Object({
            valor: 1.6 + i*2/10,
            U: 0.018 + i*2/500,
            leituras: new Array(4).fill()
          })
        ),
        amostras: [
          {
            leituras: new Array(4).fill()
          }
        ]
      }
    }
  },
  methods: {
    genPermalink () {
      
      function replacer(key, value) {
        if (key === '$sheet') {
          return undefined;
        }
        return value;
      }

      this.permalink = window.location.origin + window.location.pathname + '#/' + (JSON.stringify(this.state, replacer))
      this.$nextTick(() => {
        this.$refs.permalinkHref.click()
      })
    },
    meanAndSd (item) {
      let arr = item.leituras
        .map(parseFloat)
        .filter(x => !isNaN(x))

      let n = arr.length

      let mean = arr
        .reduce((a, c) => a+parseFloat(c), 0) / arr.length

      let sd = Math.sqrt(arr.map(x => Math.pow(x - mean, 2)).reduce((a, c) => a + c, 0) / (n-1))
      
      return { mean: mean, sd: sd }
    }
  },
  computed: {
    mrcsOrdem () {
      return JSON.parse(JSON.stringify(this.state.mrcs))
        .sort((a, b) => a.valor - b.valor)
    },
    mrcsStat () { 
      return this.mrcsOrdem.map(this.meanAndSd)
    },
    amostrasStat () { return this.state.amostras.map(this.meanAndSd) },
    kLinha () {
      const dados = this.mrcsStat.map((item, i) => {
        return Object({
          mediaDasLeituras: item.mean,
          valorMrc: this.state.mrcs[i].valor,
        })
      })      
      
      // =(T12-T17)/(Q12-Q17)
      return (dados[0].mediaDasLeituras - dados[1].mediaDasLeituras) /
        (dados[0].valorMrc - dados[1].valorMrc)
    },
    mrcDeValorMenorQuePh () {
      return parseFloat(this.state.mrcs[0].valor)
    },
    mrcDeValorMaiorQuePh () {
      return parseFloat(this.state.mrcs[this.state.mrcs.length-1].valor)
    },
    valorMedidoDePh () {
      // =G14-(G43-G54)/E67
      return this.mrcDeValorMaiorQuePh - (this.mrcsStat[1].mean - this.amostrasStat[0].mean) / this.kLinha
    },
    calculosKLinha () {
      let coefSense1 = 1/(this.mrcDeValorMaiorQuePh-this.mrcDeValorMenorQuePh)
      
      let coefSense2 = (this.mrcsStat[this.mrcsStat.length-1].mean - this.mrcsStat[0].mean) /
        Math.pow(this.mrcDeValorMaiorQuePh - this.mrcDeValorMenorQuePh, 2)

      return [
        ['Variabilidade de E(0)',    this.mrcsStat[this.mrcsStat.length-1].sd, 2,            coefSense1  ],
        ['Resolução de E(0)',        0.05, Math.sqrt(3), coefSense1  ],
        ['Variabilidade de E(sens)', this.mrcsStat[0].sd, 2,            -coefSense1 ],
        ['Resolução de E(sens)',     0.05, Math.sqrt(3), -coefSense1 ],
        ['Junta de ref',             0.6, Math.sqrt(3), coefSense1  ],
        ['Termometro',               0.083, Math.sqrt(3), 0.198421    ],
        ['Estab. Temp',              0.1, Math.sqrt(3), 0.198421    ],
        ['pH(sens)',                 parseFloat(this.mrcsOrdem[0].U), 2,            coefSense2  ],
        ['pH(0)',                    parseFloat(this.mrcsOrdem[this.mrcsOrdem.length-1].U), 2,            -coefSense2 ],
      ]
    }


  }
}
</script>


<style scoped>

</style>
