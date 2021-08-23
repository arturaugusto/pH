<template>

<!--   <a class="button mb-6 is-small" @click="genPermalink">
    Permalink
  </a>
  <a v-show="false" ref="permalinkHref" class="button mr-2 _is-small" target="_blank" rel="noopener noreferrer" :href="permalink"></a> -->

  <section class="section is-medium has-background-link-dark">
    <h1 class="title has-text-primary-light">Calculadora pH</h1>
    <h2 class="subtitle has-text-grey-lighter ">
      Uma calculadora para realizar a estimativa de incertezas de medidas de pH.
    </h2>
  </section>

  <section class="section has-background-white-ter">
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
  </section>
  <section class="section">

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
  </section>


  <section class="section"
    id="confOkEl"
    v-bind:class="{'has-background-warning': !state.confOk, 'has-background-success': state.confOk}">
    
    <h4 class="title is-4">Configure seu instrumento para realizar medições em tensão elétrica (mV)</h4>

    <label class="checkbox">
      <input type="checkbox" v-model="state.confOk" @click="scrollParaEstaSecao">
      {{state.confOk ? 'Siga as próximas etapas' : 'clique quando estiver pronto'}}
      
    </label>
  </section>

  <div v-if="state.confOk">
    <section
      class="section" v-for="(mrc, mrc_i) in state.mrcs" :key="mrc_i"
      v-bind:class="{'has-background-white-ter': (mrc_i % 2)}"
      >
      <h4 class="title is-4">Insira o eletrodo no MRC de {{mrc.valor}} pH e colete 4 leituras</h4>
      <div class="field has-addons mr-2" v-for="(leitura, leitura_i) in mrc.leituras" :key="leitura_i">
        <div class="control">
          <a class="button is-static">
            leitura {{leitura_i+1}}:
          </a>
        </div>
        <div class="control">
          <input class="input" type="number" :placeholder="'leitura ' + (leitura_i+1)" v-model="mrc.leituras[leitura_i]" style="width: 120px;">
        </div>
        <div class="control">
          <a class="button is-static">
            mV
          </a>
        </div>
      </div>
      <p>valor médio: {{parseFloat(mrcsStat[mrc_i].mean.toFixed(5))}}</p>
    </section>

    <section
      class="section" v-for="(amostra, amostra_i) in state.amostras" :key="amostra_i">
      <h4 class="title is-4">Insira o eletrodo na amostra e colete 4 leituras</h4>
        <div class="field has-addons mr-2" v-for="(leitura, leitura_i) in amostra.leituras" :key="leitura_i">
          <div class="control">
            <a class="button is-static">
              leitura {{leitura_i+1}}:
            </a>
          </div>
          <div class="control">
            <input class="input" type="number" :placeholder="'leitura ' + (leitura_i+1)" v-model="amostra.leituras[leitura_i]" style="width: 120px;">
          </div>
          <div class="control">
            <a class="button is-static">
              mV
            </a>
          </div>
        </div>
      <!-- {{amostrasStat}} -->
    </section>


    <section 
    class="section"
    v-bind:class="{'has-background-danger': !resultadoValido, 'has-background-success-light': resultadoValido}"
    >

      <h3 v-if="!resultadoValido" class="title is-3">Erro. Verifique se todos os campos foram preenchidos corretamente.</h3>
      <h3 v-if="resultadoValido" class="title is-3">Resultado: {{ valorMedidoDePh }} pH ± {{ calculosPhxU }}</h3>

      <div class="control">
        <a class="button is-info" @click="mostarCalculosDetalhados = !mostarCalculosDetalhados">
          {{!mostarCalculosDetalhados ? 'Mostrar balanço de incertezas' : 'Esconder balanço de incertezas'}}
        </a>
      </div>      
    </section>

    <section v-if="state.confOk && mostarCalculosDetalhados" class="section has-background-primary-light">
      <div>

        <div class="columns">
          <div class="column">
            <table class="table is-striped is-bordered">
              <thead class="has-background-grey-light">
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
                <tr v-for="(row, row_i) in calculosKLinha" :key="row[0]">
                  <td v-for="(col, col_i) in row" :key="col_i">{{col_i > 0 ? col.toFixed(3) : col}}</td>
                  <td>{{calculosKLinhaIncPad[row_i].toFixed(3)}}</td>
                  <!-- <td v-bind:style="{ color: calculosKLinhaIncPorcentoColor[row_i]}">{{calculosKLinhaIncPorcento[row_i]}}</td> -->
                  <td>{{(calculosKLinhaIncPorcento[row_i]).toFixed(0)}}</td>
                </tr>
              </tbody>
            </table>

            <div class="mb-6"><strong>u(k'): {{calculosKLinhaU.toFixed(4)}}</strong></div>

          </div>
          <div class="column">
            <table class="table is-striped is-bordered">
              <thead class="has-background-grey-light">
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
                <tr v-for="(row, row_i) in calculosPhx" :key="row[0]">
                  <td v-for="(col, col_i) in row" :key="col_i">{{col_i > 0 ? (parseFloat(col)).toFixed(3) : col}}</td>
                  <td>{{calculosPhxIncPad[row_i].toFixed(6)}}</td>
                  <td>{{calculosPhxIncPorcento[row_i].toFixed(0)}}</td>
                </tr>
              </tbody>
            </table>

            <div class="mb-6"><strong>U: {{calculosPhxU}} pH</strong></div>

          </div>
        </div>

      </div>
    </section>
  </div>


  <section v-if="state.confOk" class="section has-background-white-ter">
    <h4 class="title is-4">Esta calculadora preenchida poderá ser acessada através do link:</h4>

    <div class="field has-addons mr-2">
      <div class="control">
        <a class="button is-static">
          Endereço:
        </a>
      </div>
      <div class="control">
        <input class="input" style="width: 40vw;" :value="permalink">
      </div>
      <div class="control">
        <a ref="permalinkHref" class="button is-link" target="_blank" rel="noopener noreferrer" :href="permalink">Abrir</a>
      </div>
    </div>

    <p class="m-2"><strong>Ou QR-Code:</strong></p>
    <div>
      <canvas ref="qrCodeCanvas"></canvas>
    </div>

  </section>


</template>

<script>

import {numberToColorHsl, toFigures, getResolution} from '@/utils.js'

import JSONCrush from '@/assets/JSONCrush.min.js'

import QRCode from 'qrcode'


export default {

  // components: {
  //   RuleNode
  // },
  name: 'HelloWorld',
  props: {
    
  },
  watch: {
    state: {
      deep: true,
      handler () {
        if (this.timeoutId) window.clearTimeout(this.timeoutId)

        this.timeoutId = window.setTimeout(() => {
          this.genPermalink()
        }, 1000)
      }
    }
  },
  mounted () {
    let urlString = (window.location.href.split('/#/').slice(1).join('/#/'))

    if (urlString.length) {
      this.state = JSON.parse(JSONCrush.uncrush(atob(decodeURIComponent(urlString))))
    }
  },
  data () {
    return {
      timeoutId: undefined,
      permalink: '',
      mostarCalculosDetalhados: false,
      state: {
        confOk: false,
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
    scrollParaEstaSecao () {
      // let hrefBkp = location.href
      // location.href = "#confOkEl"
      // this.$nextTick(() => {
      //   location.href = hrefBkp
      // })
    },
    genPermalink () {
      function replacer(key, value) {
        if (key === '$sheet') {
          return undefined;
        }
        return value;
      }
      this.permalink = window.location.origin + window.location.pathname + '#/' + btoa(JSONCrush.crush((JSON.stringify(this.state, replacer))))

      QRCode.toCanvas(this.$refs.qrCodeCanvas, this.permalink, function (error) {
        if (error) console.error(error)
      })
      // this.$nextTick(() => {
      //   this.$refs.permalinkHref.click()
      // })
    },
    meanAndSd (item) {
      let arr = item.leituras
        .map(parseFloat)
        .filter(x => !isNaN(x))

      let n = arr.length

      let mean = arr
        .reduce((a, c) => a+parseFloat(c), 0) / arr.length

      let sd = Math.sqrt(arr.map(x => Math.pow(x - mean, 2)).reduce((a, c) => a + c, 0) / (n-1))
      
      return { mean: mean, sd: sd, n: n }
    }
  },
  computed: {
    resultadoValido () {
      return !isNaN(parseFloat(this.valorMedidoDePh)) && !isNaN(parseFloat(this.calculosPhxU))
    },
    // =G14-(G43-G54)/E67
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
      let resolIncerteza = getResolution(this.calculosPhxU)
      let valor = this.mrcDeValorMaiorQuePh - (this.mrcsStat[1].mean - this.amostrasStat[0].mean) / this.kLinha
      return  valor.toFixed(resolIncerteza)
    },

    // Klinha
    calculosKLinha () {
      let coefSense1 = 1/(this.mrcDeValorMaiorQuePh-this.mrcDeValorMenorQuePh)
      
      let coefSense2 = (this.mrcsStat[this.mrcsStat.length-1].mean - this.mrcsStat[0].mean) /
        Math.pow(this.mrcDeValorMaiorQuePh - this.mrcDeValorMenorQuePh, 2)

      return [
        ['Variabilidade de E(0)',    this.mrcsStat[this.mrcsStat.length-1].sd,  2,            coefSense1  ],
        ['Resolução de E(0)',        0.05,                                      Math.sqrt(3), coefSense1  ],
        ['Variabilidade de E(sens)', this.mrcsStat[0].sd,                       2,            -coefSense1 ],
        ['Resolução de E(sens)',     0.05,                                      Math.sqrt(3), -coefSense1 ],
        ['Junta de ref',             0.6,                                       Math.sqrt(3), coefSense1  ],
        ['Termometro',               0.083,                                     Math.sqrt(3), 0.198421    ],
        ['Estab. Temp',              0.1,                                       Math.sqrt(3), 0.198421    ],
        ['pH(sens)',                 parseFloat(this.mrcsOrdem[0].U),           2,            coefSense2  ],
        ['pH(0)',                    parseFloat(this.mrcsOrdem[this.mrcsOrdem.length-1].U), 2, -coefSense2 ],
      ]
    },
    calculosKLinhaIncPad () {
      return this.calculosKLinha.map(x => Math.abs(x[1]*x[3])/x[2])
    },
    calculosKLinhaU () {
      return 2*Math.sqrt(this.calculosKLinhaIncPad.reduce((a, c) =>a+c*c, 0))
    },
    calculosKLinhaIncPadSomaIncPad () {
      return this.calculosKLinhaIncPad.reduce((a, c) => a+c,0)
    },
    calculosKLinhaIncPorcento () {
      return this.calculosKLinhaIncPad.map(x => x*100/this.calculosKLinhaIncPadSomaIncPad)
    },
    calculosKLinhaIncPorcentoColor () {
      return this.calculosKLinhaIncPorcento.map(x => numberToColorHsl(x, 0, Math.max(0, ...this.calculosKLinhaIncPorcento)))
    },
    // pHx
    calculosPhx () {
      let coefSense1 = -1/this.kLinha
      let variabilidadeE0 = this.amostrasStat[this.amostrasStat.length-1].sd/Math.sqrt(this.amostrasStat[this.amostrasStat.length-1].n)

      let coefSenseAng = 0
      if (this.amostrasStat && this.amostrasStat.length && this.mrcsStat && this.mrcsStat.length) {
        coefSenseAng = ([this.mrcsStat[1].mean - this.amostrasStat[0].mean])/Math.pow(this.kLinha, 2)
      }

      return [
        ['Variabilidade de E(x)',   variabilidadeE0,                            2,            coefSense1  ],
        ['Resolução de E(x)',       0.05,                                       Math.sqrt(3), coefSense1  ],
        ['Variabilidade de E(0)',   this.mrcsStat[this.mrcsStat.length-1].sd,   2,            coefSense1  ],
        ['Resolução de E(0)',       0.05,                                       Math.sqrt(3), coefSense1  ],
        ['Coef. Angular',           this.calculosKLinhaU,                       2,            coefSenseAng],
        ['Inc. pH(0)',              this.mrcsOrdem[this.mrcsOrdem.length-1].U,  2,            1           ],
      ]
    },
    calculosPhxIncPad () {
      return this.calculosPhx.map(x => Math.abs(x[1]*x[3])/x[2])
    },
    calculosPhxU () {
      return toFigures(2*Math.sqrt(this.calculosPhxIncPad.reduce((a, c) =>a+c*c, 0)), 2)
    },
    calculosPhxIncPadSomaIncPad () {
      return this.calculosPhxIncPad.reduce((a, c) => a+c,0)
    },
    calculosPhxIncPorcento () {
      return this.calculosPhxIncPad.map(x => x*100/this.calculosPhxIncPadSomaIncPad)
    },
    calculosPhxIncPorcentoColor () {
      return this.calculosPhxIncPorcento.map(x => numberToColorHsl(x, 0, 100))
    }
  }
}
</script>


<style scoped>

</style>
