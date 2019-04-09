<template>
  <div class="hello">
    <h1>{{ title }}</h1>
    <h3>
      Escala para la Frecuencia de Fallas (filas):
      <input type="text" v-model="filas">
    </h3>
    <br>
    <br>
    <table border="1" align="center">
      <caption>Carga de Frecuencias de Fallas</caption>
      <thead>
        <tr>
          <th rowspan="2">Fila</th>
          <th colspan="3">Frecuencia de Fallas</th>
        </tr>
        <tr>
          <th>Valor</th>
          <th>Denominación</th>
          <th>Descripción</th>
        </tr>
      </thead>
      <tr v-for="(row, rowIndex) in probabilidades">
        <td>{{ rowIndex + 1}}</td>
        <td class="input">
          <input type="text" v-model="row.value" size="2">
        </td>
        <td class="input">
          <input type="text" v-model="row.denomination" size="5">
        </td>
        <td class="input">
          <input type="text" v-model="row.description" size="100">
        </td>
      </tr>
    </table>
    <h3>
      Escala para las consecuencias (columnas):
      <input type="text" v-model="columnas">
    </h3>
    <br>
    <br>
    <table border="1" align="center">
      <caption>Carga de Consecuencias (Factores de Impacto)</caption>
      <thead>
        <tr>
          <th rowspan="3">Fila</th>
          <th :colspan="getColSpan">Consecuencias</th>
        </tr>
        <tr>
          <th colspan="3" v-for="(row, rowIndex) in consecuencias">{{ row }}</th>
        </tr>
        <tr>
          <template v-for="(row, rowIndex) in consecuencias">
            <th>Valor</th>
            <th>Denominación</th>
            <th>Descripción</th>
          </template>
        </tr>
      </thead>
      <tr v-for="(row, rowIndex) in consValues">
        <td>{{ rowIndex + 1}}</td>
        <template v-for="(col, colIndex) in row">
          <template v-if="colIndex==0">
            <td class="input">
              <input type="text" v-model="col.value" size="2">
            </td>
            <td class="input">
              <input type="text" v-model="col.denomination" size="4">
            </td>
          </template>
          <template v-else>
            <td class="input">{{col.value}}</td>
            <td class="input">{{col.denomination}}</td>
          </template>
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
          :class="['celda', nivelCriticidad((probabilidades.length-1) - rowIndex, col)+'-class']"
        >{{ (probabilidades[(probabilidades.length-1) - rowIndex].value + '*' + col) + ' = ' + nivelCriticidad((probabilidades.length-1) - rowIndex, col) }}</td>
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
  name: "HelloWorld",
  props: {
    msg: String
  },
  data() {
    return {
      title: "Calibración de Matriz de Priorización por Riesgo",
      filas: 0,
      columnas: 0,
      muyAlta: 23,
      alta: 12,
      media: 5,
      probabilidades: [],
      consecuencias: ["CM", "LC", "SHA", "Imagen"],
      consValues: []
    };
  },
  methods: {
    nivelCriticidad: function(rowValue, colValue) {
      //console.log(row, col);

      let ret = "B";

      let value = this.probabilidades[rowValue].value * colValue;
      console.log(this.probabilidades[rowValue].value, colValue, value);

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
        }
        console.log(this.probabilidades);
      } else {
        // Probabilidades
        for (paso = 0; paso < this.filas; paso++) {
          if (!this.probabilidades[paso]) {
            let probObj = {};
            probObj.value = paso + 1;
            probObj.denomination = "¿?";
            probObj.description = "¿?";

            this.probabilidades.push(probObj);
          }
        }
        console.log(this.probabilidades);
      }
    },
    reDimentionCons(newColumnas, oldColumnas) {
      let paso = 0;
      let cons = 0;
      console.log(newColumnas, oldColumnas);

      if (newColumnas < oldColumnas && this.consValues.length > 0) {
        for (paso = newColumnas; paso < oldColumnas; paso++) {
          this.consValues.pop();
        }
        console.log(this.consValues);
      } else {
        for (paso = 0; paso < newColumnas; paso++) {
          if (!this.consValues[paso]) {
            let factores = [];
            for (cons = 0; cons < this.consecuencias.length; cons++) {
              let probObj = {};
              probObj.consecuencia = this.consecuencias[cons];
              probObj.denomination = "¿?";
              probObj.value = paso + 1;
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
    this.columnas = 8;
  },
  computed: {
    getColSpan() {
      return 3 * this.consecuencias.length;
    },
    arrayFilas() {
      let ret = [];
      for (let i = 1; i <= this.columnas; i++) {
        ret.push(i);
      }

      return ret;
    }
  },
  watch: {
    filas: function(newFilas, oldFilas) {
      console.log(`filas changing from ${oldFilas} to ${newFilas}`);
      this.reDimentionProbs(newFilas, oldFilas);
    },
    columnas: function(newColumnas, oldColumnas) {
      console.log(`columnas changing from ${oldColumnas} to ${newColumnas}`);
      this.reDimentionCons(newColumnas, oldColumnas);
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
