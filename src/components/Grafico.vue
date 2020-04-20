<template>
    <div class="app-container">
        <table>
            <th>
                <label for="dia">Día:</label>
                <input type="date" name="dia" id="dia" @input="actualizarPorDia()">
            </th>
            <th>
                <label for="semana">Semana:</label>
                <input type="date" name="semana" id="semana" @input="actualizarPorSemana()">
            </th>
            <th>
                <label for="mes">Mes:</label>
                <input type="date" name="mes" id="mes" @input="actualizarPorMes()">
            </th>
            <th>
                <label for="anio">Año:</label>
                <input type="date" name="anio" id="anio" @input="actualizarPorAnio()">
            </th>
        </table>

        <div class="lapso-container">
            <p>{{texto_lapso_de_muestra}}</p>
        </div>

        <div class="variable-container">
            <p>{{variable_seleccionada}}</p>
        </div>

        <div class="chart-container">
            <Chart :chart-data="datacollection"></Chart>
        </div>


        <table id="table-variables" align="center">
            <tr>
                <td @click="cargarDatosEnGrafico(variables)"
                    v-for="variables in tableVariables1"
                    :key="variables">
                {{ variables }}
                </td>
            </tr>
            <tr>
                <td @click="cargarDatosEnGrafico(variables)"
                    v-for="variables in tableVariables2"
                    :key="variables">
                {{ variables }}
                </td><br>
            </tr>
        </table>


    </div>
</template>

<script>
import axios from 'axios';
import Chart from './vueChart_line.js';

export default {
    name: 'grafico_de_variables',
    components: {Chart},
    data(){
        return{
            urlApi: 'http://papvidadigital-test.com/test',
            datacollection: null,

            lista_de_nodos: null,
            lista_de_variables: null,
            
            nodo_seleccionado: null,
            variable_seleccionada: null,
            lista_variables_por_nodo: null,

            lecturas_por_nodo: null,

            tableVariables1: null,
            tableVariables2: null,
            ultimasLecturas: null,

            texto_lapso_de_muestra: null,
            
        }
    },
    mounted(){
        this.obtenerNombreDeNodos();
        this.obtenerNombreDeVariables();

        this.nodo_seleccionado = (window.location.href.includes('#'))
                                    ? this.nodo_seleccionado = window.location.href.substr(window.location.href.indexOf('#')+1) 
                                    : null;

        this.obtenerUltimaLectura();

        // this.nodo_seleccionado = window.location.href.substr(window.location.href.indexOf('#')+1);
        // console.log('->>>'+this.nodo_seleccionado);
        // console.log(this.lista_de_nodos);
        // console-log(this.lista_de_variables);
    },
    methods: {
        obtenerNombreDeNodos(){
            axios.get( this.urlApi).then( response =>{
                let ids = response.data;
                ids = ids.map( x => {return x.id});
                this.lista_de_nodos = ids;
            }).catch( e => {
                console.log(e)
            })
        },
        obtenerNombreDeVariables(){
            axios.get( this.urlApi + '/variables' ).then( response =>{
                let variables = [];
                response.data.forEach(element => variables.push({code: element.code, nombre: element.description}) );
                this.lista_de_variables = variables;
            }).catch( e => {
                console.log(e)
            })
        },
        obtenerUltimaLectura(){
            axios.get( this.urlApi + '/' + this.nodo_seleccionado + '/lastread' ).then( response => {
                // this.datacollection = null;
                // this.variable_seleccionada = null;

                let lastread = response.data;
                let lastdate = {};

                if( Object.prototype.hasOwnProperty.call(lastread, 'ts') ){
                    lastdate = lastread.ts;
                }else if(Object.prototype.hasOwnProperty.call(lastread, 'tn')){
                    lastdate = lastread.tn;
                }else{
                    lastdate = lastread.tu;
                }

                let variablesDelNodo = [];
                for( let i = 0 ; i < this.lista_de_variables.length ; i++){
                    if( Object.prototype.hasOwnProperty.call(lastread, this.lista_de_variables[i].code) ){
                        variablesDelNodo.push(this.lista_de_variables[i].nombre);
                    }
                }

                this.tableVariables1 = [];
                this.tableVariables2 = [];
                this.ultimasLecturas = [];
                for( let i = 0 ; i < variablesDelNodo.length ; i++){

                    if(i < 4 ){
                        this.tableVariables1.push(variablesDelNodo[i]); 
                    }else{
                        this.tableVariables2.push(variablesDelNodo[i]); 
                    }
                    this.ultimasLecturas.push(lastread[variablesDelNodo[i]]);
                }
                for( let i = 0 ; i < 4 ; i++){
                    if( this.tableVariables1.length < 4 ){
                        this.tableVariables1.push(null);
                    }
                    if( this.tableVariables2.length < 4 ){
                        this.tableVariables2.push(null);
                    }
                }


                axios.get(`${this.urlApi}/day/${this.nodo_seleccionado}/${parseInt(lastdate.substr(0,4))}/${parseInt(lastdate.substr(5,2))}/${parseInt(lastdate.substr(8,2))}`).then( responseofday => {
                    let total_data = responseofday.data;

                    if(total_data.length > 0)
                {
                    let nodo_variables = JSON.parse(total_data[0].data);
                    delete nodo_variables.id;
                    delete nodo_variables.ac;
                    delete nodo_variables.ts;
                    delete nodo_variables.tn;

                    this.lista_variables_por_nodo = Object.keys(nodo_variables);
                    
                    this.lecturas_por_nodo = new Array(total_data.length);
                    let counter = 0;
                    total_data.forEach(elem => {
                        let jsontemp = {};
                        jsontemp.id = elem.id;
                        jsontemp.fecha_hora = elem.fecha_hora;

                        for( let i = 0 ; i < this.lista_variables_por_nodo.length ; i++){
                            jsontemp[this.lista_variables_por_nodo[i]] = JSON.parse(elem.data)[this.lista_variables_por_nodo[i]];
                            
                        }

                        this.lecturas_por_nodo[counter] = jsontemp;
                        counter++;
                    });



                    console.log(this.lista_variables_por_nodo)
                    console.log(this.lecturas_por_nodo);
                }else{
                    this.lista_variables_por_nodo = null;
                    // alert('No hay ninguna lectura del nodo');
                }
                }).catch( e  => {
                    console.log(e);
                });
                
            }).catch( e => {
                console.log(e);
            });
        },
        cargarDatosEnGrafico(nombrevariable){

            if( nombrevariable == null) return;

            this.variable_seleccionada = nombrevariable;
            let codevariable;

            this.lista_de_variables.forEach( elem => {
                if( elem.nombre.localeCompare(nombrevariable) == 0 ) codevariable = elem.code;
            });

            let labels_lectura = this.lecturas_por_nodo.map( elem => {return elem.fecha_hora});
            let data_lectura = this.lecturas_por_nodo.map( elem => {return elem[codevariable]} );

            this.formatoLapsoTiempo();

            this.datacollection = {
                labels: labels_lectura,
                datasets: [
                    {
                        label: nombrevariable,
                        backgroundColor: 'rgba(0, 0, 0, 0)',
                        borderColor: 'aqua', 
                        pointBackgroundColor: 'blue', 
                        pointRadius: 0,
                        pointBorderColor: 'blue',
                        data: data_lectura
                    }
                ]
            };

        },
        actualizarPorDia(){
            let date = document.getElementById('dia').value;

            axios.get(`${this.urlApi}/day/${this.nodo_seleccionado}/${parseInt(date.substr(0,4))}/${parseInt(date.substr(5,2))}/${parseInt(date.substr(8,2))}`).then( response => {
                let total_data = response.data;

                if( total_data.length > 0 ){
                    let nodo_variables = JSON.parse(total_data[0].data);
                    delete nodo_variables.id;
                    delete nodo_variables.ac;
                    delete nodo_variables.ts;
                    delete nodo_variables.tn;

                    this.variables_por_nodo = Object.keys(nodo_variables);
                    
                    this.lecturas_por_nodo = new Array(total_data.length);
                    let counter = 0;
                    total_data.forEach(elem => {
                        let jsontemp = {};
                        jsontemp.id = elem.id;
                        jsontemp.fecha_hora = elem.fecha_hora;

                        for( let i = 0 ; i < this.variables_por_nodo.length ; i++){
                            jsontemp[this.variables_por_nodo[i]] = JSON.parse(elem.data)[this.variables_por_nodo[i]];
                            
                        }

                        this.lecturas_por_nodo[counter] = jsontemp;
                        counter++;
                    });

                    this.cargarDatosEnGrafico(this.variable_seleccionada);

                }else{
                    this.datacollection = {
                        labels: [],
                        datasets: []
                    };
                    // console.log(date.length);
                    if(date.length != 0){
                        alert('No hay ninguna lectura del nodo');
                    }
                }
            }).catch( e => {
                console.log(e);
            });

        },
        actualizarPorSemana(){
            let date = document.getElementById('semana').value;

            axios.get(`${this.urlApi}/week/${this.nodo_seleccionado}/${parseInt(date.substr(0,4))}/${parseInt(date.substr(5,2))}/${parseInt(date.substr(8,2))}`).then( response => {
                let total_data = response.data;

                if( total_data.length > 0 ){
                    let nodo_variables = JSON.parse(total_data[0].data);
                    delete nodo_variables.id;
                    delete nodo_variables.ac;
                    delete nodo_variables.ts;
                    delete nodo_variables.tn;

                    this.variables_por_nodo = Object.keys(nodo_variables);
                    
                    this.lecturas_por_nodo = new Array(total_data.length);
                    let counter = 0;
                    total_data.forEach(elem => {
                        let jsontemp = {};
                        jsontemp.id = elem.id;
                        jsontemp.fecha_hora = elem.fecha_hora;

                        for( let i = 0 ; i < this.variables_por_nodo.length ; i++){
                            jsontemp[this.variables_por_nodo[i]] = JSON.parse(elem.data)[this.variables_por_nodo[i]];
                            
                        }

                        this.lecturas_por_nodo[counter] = jsontemp;
                        counter++;
                    });

                    this.cargarDatosEnGrafico(this.variable_seleccionada);

                }else{
                    this.datacollection = {
                        labels: [],
                        datasets: []
                    };
                    // console.log(date.length);
                    if(date.length != 0){
                        alert('No hay ninguna lectura del nodo');
                    }
                }
            }).catch( e => {
                console.log(e);
            });
        },
        actualizarPorMes(){
            let date = document.getElementById('mes').value;

            axios.get(`${this.urlApi}/month/${this.nodo_seleccionado}/${parseInt(date.substr(0,4))}/${parseInt(date.substr(5,2))}`).then( response => {
                let total_data = response.data;

                if( total_data.length > 0 ){
                    let nodo_variables = JSON.parse(total_data[0].data);
                    delete nodo_variables.id;
                    delete nodo_variables.ac;
                    delete nodo_variables.ts;
                    delete nodo_variables.tn;

                    this.variables_por_nodo = Object.keys(nodo_variables);
                    
                    this.lecturas_por_nodo = new Array(total_data.length);
                    let counter = 0;
                    total_data.forEach(elem => {
                        let jsontemp = {};
                        jsontemp.id = elem.id;
                        jsontemp.fecha_hora = elem.fecha_hora;

                        for( let i = 0 ; i < this.variables_por_nodo.length ; i++){
                            jsontemp[this.variables_por_nodo[i]] = JSON.parse(elem.data)[this.variables_por_nodo[i]];
                            
                        }

                        this.lecturas_por_nodo[counter] = jsontemp;
                        counter++;
                    });

                    this.cargarDatosEnGrafico(this.variable_seleccionada);

                }else{
                    this.datacollection = {
                        labels: [],
                        datasets: []
                    };
                    // console.log(date.length);
                    if(date.length != 0){
                        alert('No hay ninguna lectura del nodo');
                    }
                }
            }).catch( e => {
                console.log(e);
            });
        },
        actualizarPorAnio(){
            let date = document.getElementById('anio').value;

            axios.get(`${this.urlApi}/year/${this.nodo_seleccionado}/${parseInt(date.substr(0,4))}`).then( response => {
                let total_data = response.data;

                if( total_data.length > 0 ){
                    let nodo_variables = JSON.parse(total_data[0].data);
                    delete nodo_variables.id;
                    delete nodo_variables.ac;
                    delete nodo_variables.ts;
                    delete nodo_variables.tn;

                    this.variables_por_nodo = Object.keys(nodo_variables);
                    
                    this.lecturas_por_nodo = new Array(total_data.length);
                    let counter = 0;
                    total_data.forEach(elem => {
                        let jsontemp = {};
                        jsontemp.id = elem.id;
                        jsontemp.fecha_hora = elem.fecha_hora;

                        for( let i = 0 ; i < this.variables_por_nodo.length ; i++){
                            jsontemp[this.variables_por_nodo[i]] = JSON.parse(elem.data)[this.variables_por_nodo[i]];
                            
                        }

                        this.lecturas_por_nodo[counter] = jsontemp;
                        counter++;
                    });

                    this.cargarDatosEnGrafico(this.variable_seleccionada);

                }else{
                    this.datacollection = {
                        labels: [],
                        datasets: []
                    };
                    // console.log(date.length);
                    if(date.length != 0){
                        alert('No hay ninguna lectura del nodo');
                    }
                }
            }).catch( e => {
                console.log(e);
            });
        },
        formatoLapsoTiempo(){ //Asigna el valor de lapso de muestra con formato
            let inicio = this.lecturas_por_nodo[0].fecha_hora;
            let fin = this.lecturas_por_nodo[this.lecturas_por_nodo.length-1].fecha_hora;

            switch( inicio.substr(5,2) ){
                case '01': inicio = `${inicio.substr(8,2)} de Enero del ${inicio.substr(0,4)}`; break;
                case '02': inicio = `${inicio.substr(8,2)} de Febrero del ${inicio.substr(0,4)}`; break;
                case '03': inicio = `${inicio.substr(8,2)} de Marzo del ${inicio.substr(0,4)}`; break;
                case '04': inicio = `${inicio.substr(8,2)} de Abril del ${inicio.substr(0,4)}`; break;
                case '05': inicio = `${inicio.substr(8,2)} de Mayo del ${inicio.substr(0,4)}`; break;
                case '06': inicio = `${inicio.substr(8,2)} de Junio del ${inicio.substr(0,4)}`; break;
                case '07': inicio = `${inicio.substr(8,2)} de Julio del ${inicio.substr(0,4)}`; break;
                case '08': inicio = `${inicio.substr(8,2)} de Agosto del ${inicio.substr(0,4)}`; break;
                case '09': inicio = `${inicio.substr(8,2)} de Septiembre del ${inicio.substr(0,4)}`; break;
                case '10': inicio = `${inicio.substr(8,2)} de Octubre del ${inicio.substr(0,4)}`; break;
                case '11': inicio = `${inicio.substr(8,2)} de Noviembre del ${inicio.substr(0,4)}`; break;
                case '12': inicio = `${inicio.substr(8,2)} de Diciembre del ${inicio.substr(0,4)}`; break;
            }

            switch( fin.substr(5,2) ){
                case '01': fin = `${fin.substr(8,2)} de Enero del ${fin.substr(0,4)}`; break;
                case '02': fin = `${fin.substr(8,2)} de Febrero del ${fin.substr(0,4)}`; break;
                case '03': fin = `${fin.substr(8,2)} de Marzo del ${fin.substr(0,4)}`; break;
                case '04': fin = `${fin.substr(8,2)} de Abril del ${fin.substr(0,4)}`; break;
                case '05': fin = `${fin.substr(8,2)} de Mayo del ${fin.substr(0,4)}`; break;
                case '06': fin = `${fin.substr(8,2)} de Junio del ${fin.substr(0,4)}`; break;
                case '07': fin = `${fin.substr(8,2)} de Julio del ${fin.substr(0,4)}`; break;
                case '08': fin = `${fin.substr(8,2)} de Agosto del ${fin.substr(0,4)}`; break;
                case '09': fin = `${fin.substr(8,2)} de Septiembre del ${fin.substr(0,4)}`; break;
                case '10': fin = `${fin.substr(8,2)} de Octubre del ${fin.substr(0,4)}`; break;
                case '11': fin = `${fin.substr(8,2)} de Noviembre del ${fin.substr(0,4)}`; break;
                case '12': fin = `${fin.substr(8,2)} de Diciembre del ${fin.substr(0,4)}`; break;
            }

            this.texto_lapso_de_muestra = `${inicio} al ${fin}`


        }



    }
}
</script>

<style>

.chart-container{
    width: 500px;
    height: auto;
    margin: 20px auto;
}
table{
    width: 100%;
    height: 100px;
}
p{
    color: aliceblue;
}
hr{
    opacity: 50%;
}
label{
    color: whitesmoke;
}
/* .app-container{
    background: black;
} */
.app-container{
    background: rgb(28,29,33);
    background: radial-gradient(circle, rgba(28,29,33,1) 0%, rgba(33,34,37,1) 36%, rgba(0,0,0,1) 100%);

}

#table-variables tr:first-child td{ border-top: 0;}
#table-variables tr td:first-child{ border-left: 0;}
#table-variables tr:last-child td{ border-bottom: 0;}
#table-variables tr td:last-child{ border-right: 0;}

#table-variables{
    width: 752px;
    height: 360px;
    border-collapse: collapse;
    color: aliceblue;
    font: bold;
    font-size: x-large;
}
#table-variables button{
    border-radius: 12px;
}
/* filas */
#table-variables tr td{ 
    border: 2px solid rgba(221,221,221,0.15);
    height: 168px;
    width: 182px;
}
/* columnas */
#table-variables tr{
    height: 168px;
    width: 182px;
}

</style>