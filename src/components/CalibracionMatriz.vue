<template>
  <div class="hello">
    <h1>{{ title }}</h1>
    <h3>
      Filas:
      <input type="text" v-model="filas">
    </h3>
    <br>
    <br>
    <table border="1" align="center">
      <caption>Carga de Frecuencias y Consecuencias</caption>
      <thead>
        <tr>
          <th rowspan="3">Fila</th>
          <th colspan="2" rowspan="2">Frecuencia de Fallas</th>
          <th :colspan="getColSpan">Consecuencias</th>
        </tr>
        <tr>
          <th colspan="2" v-for="(row, rowIndex) in consecuencias">{{ row }}</th>
        </tr>
        <tr>
          <th>Valor</th>
          <th>Descripción</th>
          <template v-for="(row, rowIndex) in consecuencias">
            <th>Valor</th>
            <th>Descripción</th>
          </template>
        </tr>
      </thead>
      <tr v-for="(row, rowIndex) in probabilidades">
        <td>{{ rowIndex + 1}}</td>
        <td class="input">
          <input type="text" v-model="row.value" size="2">
        </td>
        <td class="input">
          <input type="text" v-model="row.description">
        </td>
        <template v-for="(col, colIndex) in consValues[rowIndex]">
          <td class="input">
            <input type="text" size="2" v-model="col.value">
          </td>
          <td class="input">
            <input type="text" v-model="col.description">
          </td>
        </template>
      </tr>
    </table>
    <h3>Niveles de Criticidad</h3>
    <div>
      Muy Alta:
      <input type="text" size="2" v-model="muyAlta">
      <br>Alta:
      <input type="text" size="2" v-model="alta">
      <br>Media:
      <input type="text" size="2" v-model="media">
      <br>
    </div>
    <br>
    <table border="1" align="center">
      <caption>Matriz de Priorización por Riesgo</caption>
      <tr v-for="(row, rowIndex) in probabilidades">
        <td class="escala celda">{{ probabilidades.length - rowIndex }}</td>
        <td
          v-for="(col, colIndex) in arrayFilas"
          :class="['celda', nivelCriticidad((probabilidades.length - rowIndex)*col)+'-class']"
        >{{ nivelCriticidad((probabilidades.length - rowIndex)*col) }}</td>
      </tr>
      <tr class="escala">
        <td></td>
        <td v-for="(col, colIndex) in arrayFilas">{{ col }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: "CalibracionMatriz",
  props: {
    msg: String
  },
  data() {
    return {
      title: "Calibración de Matriz de Priorización por Riesgo",
      filas: 0,
      muyAlta: 15,
      alta: 10,
      media: 5,
      probabilidades: [],
      consecuencias: ["CM", "LC", "SHA", "Imagen"],
      consValues: []
    };
  },
  methods: {
    nivelCriticidad(value) {
      let ret = "B";

      if (value >= this.muyAlta) ret = "MA";
      else if (value >= this.alta) ret = "A";
      else if (value >= this.media) ret = "M";

      return ret;
    },
    reDimentionProbs(newFilas, oldFilas) {
      let paso = 0;
      let probs = 0;
      console.log(newFilas, oldFilas);

      if (newFilas < oldFilas && this.probabilidades.length > 0) {
        for (paso = newFilas; paso < oldFilas; paso++) {
          this.probabilidades.pop();
          this.consValues.pop();
        }
        console.log(this.probabilidades);
        console.log(this.consValues);
      } else {
        // Probabilidades
        for (paso = 0; paso < this.filas; paso++) {
          //for (paso = this.filas; paso > 0; paso--) {
          if (!this.probabilidades[paso]) {
            let probObj = {};
            probObj.value = paso + 1;
            probObj.description = "¿?";

            this.probabilidades.push(probObj);
          }
        }
        console.log(this.probabilidades);

        for (probs = 0; probs < this.filas; probs++) {
          if (!this.consValues[probs]) {
            let factores = [];
            for (paso = 0; paso < this.consecuencias.length; paso++) {
              let probObj = {};
              probObj.consecuencia = this.consecuencias[paso];
              probObj.value = probs + 1;
              probObj.description = "¿?";

              factores.push(probObj);
            }
            this.consValues.push(factores);
          }
        }
        console.log(this.consValues);
      }
    }
  },
  created() {
    this.filas = 5;
  },
  computed: {
    getColSpan() {
      return 2 * this.consecuencias.length;
    },
    arrayFilas() {
      let ret = [];
      for (let i = 1; i <= this.filas; i++) {
        ret.push(i);
      }

      return ret;
    }
  },
  watch: {
    filas: function(newFilas, oldFilas) {
      console.log(`filas changing from ${oldFilas} to ${newFilas}`);
      this.reDimentionProbs(newFilas, oldFilas);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.input {
  padding: 5px;
}
.escala {
  background: #000;
  color: #fff;
}
.celda {
  padding: 10px;
}
.MA-class {
  background: red;
  color: white;
}
.A-class {
  background: green;
  color: white;
}
.M-class {
  background: yellow;
}
</style>
