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
            <!-- :options="dataOptions"></Chart> -->
        </div>


        <table id="table-variables" align="center">
            <tr>

                
                <td @click="cargarDatosEnGrafico(variables.nombre)"
                    v-on:click="pushed"
                    v-for="variables in tableVariables1"
                    :key="variables.id">

                <div class="celda-variable-nombre">{{ variables.nombre }}</div>
                <div class="celda-variable-lectura">{{ variables.lectura}}</div>
                
                </td>
            </tr>
            <tr>

                <td @click="cargarDatosEnGrafico(variables.nombre)"
                    v-on:click="pushed"
                    v-for="variables in tableVariables2"
                    :key="variables.id">
                <div class="celda-variable-nombre">{{ variables.nombre }}</div>
                <div class="celda-variable-lectura">{{ variables.lectura}}</div>
                </td>

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
            // dataOptions: null,

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
            labels_lapso: null,
            tipo_de_lapso: 'dia',
            
        }
    },
    mounted(){
        this.obtenerNombreDeNodos();
        this.obtenerNombreDeVariables();



        // this.nodo_seleccionado = (window.location.href.includes('#'))
        //                             ? this.nodo_seleccionado = window.location.href.substr(window.location.href.indexOf('#')+1) 
        //                             : null;

        // this.obtenerUltimaLectura();

        // this.nodo_seleccionado = window.location.href.substr(window.location.href.indexOf('#')+1);
        // console.log('->>>'+this.nodo_seleccionado);
        // console.log(this.lista_de_nodos);
        // console-log(this.lista_de_variables);

        // this.dataOptions = {
        //     legend: {
        //         labels: {
        //         // This more specific font property overrides the global property
        //         fontColor: 'yellow'
        //         }
        //     }
        // }
        

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

                this.nodo_seleccionado = (window.location.href.includes('#'))
                                    ? this.nodo_seleccionado = window.location.href.substr(window.location.href.indexOf('#')+1) 
                                    : null;

                this.obtenerUltimaLectura();
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
                let codeVariablesDelNodo = [];
                for( let i = 0 ; i < this.lista_de_variables.length ; i++){
                    if( Object.prototype.hasOwnProperty.call(lastread, this.lista_de_variables[i].code) ){
                        variablesDelNodo.push(this.lista_de_variables[i].nombre);
                        codeVariablesDelNodo.push(this.lista_de_variables[i].code);
                    }
                }

                this.tableVariables1 = [];
                this.tableVariables2 = [];
                this.ultimasLecturas = [];
                
                for( let i = 0 ; i < variablesDelNodo.length ; i++){

                    let tempJson = {};

                    if(i < 4 ){
                        tempJson.id = i;
                        tempJson.nombre = variablesDelNodo[i];
                        tempJson.lectura = lastread[codeVariablesDelNodo[i]]
                        this.tableVariables1.push(tempJson);
                        // this.tableVariables1.push(variablesDelNodo[i]); 
                    }else{
                        tempJson.id = i;
                        tempJson.nombre = variablesDelNodo[i];
                        tempJson.lectura = lastread[codeVariablesDelNodo[i]]
                        this.tableVariables2.push(tempJson);
                        // this.tableVariables2.push(variablesDelNodo[i]); 
                    }
                    
                    // tempJson[variablesDelNodo[i]] = lastread[codeVariablesDelNodo[i]];
                    // this.ultimasLecturas.push( tempJson );
                    // console.log(lastread[])
                    // console.log(variablesDelNodo);
                }
                
                for( let i = 0 ; i < 4 ; i++){
                    if( this.tableVariables1.length < 4 ){
                        this.tableVariables1.push({nombre: ' ' , lectura: null});
                    }
                    if( this.tableVariables2.length < 4 ){
                        this.tableVariables2.push({nombre: ' ' , lectura: null});
                    }
                }

                // console.table(this.tableVariables1);
                // console.log('----------------')
                // console.log(this.ultimasLecturas);
                // console.log('ultima lectura');
                // console.log(this.tableVariables1[0].nombre);

                // this.cargarDatosEnGrafico(this.tableVariables1[0].nombre);
                this.tipo_de_lapso = 'dia';



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


                    this.cargarDatosEnGrafico(this.tableVariables1[0].nombre);
                    // console.log('buen');
                    // console.log(this.lecturas_por_nodo);

                    // console.log('testing');


                    // console.log(self.document.getElementById('table-variables').rows[0].children[0]);
                    
                    document.getElementById('table-variables').rows[0].children[0].style.background = 'radial-gradient(circle, rgba(26,33,54,1) 0%, rgba(101,101,103,1) 82%)'

                    
                    // this.cargarDatosEnGrafico(this.lista_variables_por_nodo[0]);
                    // console.log('buen test');
                    // console.log(this.lista_variables_por_nodo[0]);


                    // console.log(this.lista_variables_por_nodo);
                    // console.log(this.lecturas_por_nodo);
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
            if( nombrevariable.localeCompare(' ') == 0) return;

            this.variable_seleccionada = nombrevariable;
            let codevariable;

            this.lista_de_variables.forEach( elem => {
                if( elem.nombre.localeCompare(nombrevariable) == 0 ) codevariable = elem.code;
            });

            this.labels_lapso = this.lecturas_por_nodo.map( elem => {return elem.fecha_hora});

            // let labels_lectura = this.lecturas_por_nodo.map( elem => {return elem.fecha_hora});

            let data_lectura = this.lecturas_por_nodo.map( elem => {return elem[codevariable]} );

            this.formatoLapsoTiempo();

            this.datacollection = {
                labels: this.labels_lapso,
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
            this.tipo_de_lapso = 'dia';

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
            this.tipo_de_lapso = 'semana';

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
            this.tipo_de_lapso = 'mes';

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
            this.tipo_de_lapso = 'anio';

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

            if( this.tipo_de_lapso.localeCompare('dia') == 0 ){
                this.texto_lapso_de_muestra = inicio;
            }else{
                

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

            // 2020-02-18T06:05:05
            //Labels data formating
            // let fecheTemporal = new Date(this.labels_lapso[0].substr(0,19));
            // let weekday = fecheTemporal.getDay();

            // switch( weekday ){
            //     case 1: return 'Lunes'; break;
            //     case 2: return 'Martes'; break;
            //     case 3: return 'Miercoles'; break;
            //     case 4: return 'Jueves'; break;
            //     case 5: return 'Viernes'; break;
            //     case 6: return 'Sabado'; break;
            //     case 0: return 'Domingo'; break;
            // }


            // console.log(fecheTemporal);
            // console.log(fecheTemporal.getDay());

            let tempDate;
            let tempweekday;
            // console.log(this.labels_lapso);

            if( this.tipo_de_lapso.localeCompare('dia') == 0){
                // console.log('entro al formato');
                this.labels_lapso = this.labels_lapso.map(elem => `${elem.substr(11,8)}`);
                // this.labels_lapso = this.labels_lapso.map(elem => `${elem.substr(0,10)}`);
            }else if( this.tipo_de_lapso.localeCompare('semana') == 0){
                this.labels_lapso = this.labels_lapso.map(elem => {

                    tempDate = new Date(elem.substr(0,19));
                    tempweekday = tempDate.getDay();

                    switch( tempweekday ){
                        case 1: return 'Lunes '+ elem.substr(17,2);    
                        case 2: return 'Martes '+ elem.substr(17,2);   
                        case 3: return 'Miercoles '+ elem.substr(17,2);
                        case 4: return 'Jueves '+ elem.substr(17,2);   
                        case 5: return 'Viernes '+ elem.substr(17,2);  
                        case 6: return 'Sabado '+ elem.substr(17,2);   
                        case 0: return 'Domingo '+ elem.substr(17,2);  
                }
                    
                    });
            }else{
                this.labels_lapso = this.labels_lapso.map(elem => `${elem.substr(0,10)}`);
            }

            

            

            


        },
        pushed(){
            // console.log(event.target.parentNode.tagName);
            // console.log(event.target.parentNode.innerText);

            // console.log(event.target.parentNode.style.background);
            // if( event.target.parentNode.style.background.localeCompare('radial-gradient(circle, rgb(26, 33, 54) 0%, rgb(101, 101, 103) 82%)') == 0){
            //     event.target.parentNode.style.background = '';
            // }else{
            //     // event.target.parentNode.style.backgroundColor = "rgb(101, 101, 103)";
            //     console.log('entro');
            //     event.target.parentNode.style.background = 'radial-gradient(circle, rgba(26,33,54,1) 0%, rgba(101,101,103,1) 82%)'
            // }

            // console.log(event.target.parentNode);
            
            // console.log(event.target.parentNode.style.backgroundColor);
            // console.log(event.target.parentNode.parentNode.parentNode.rows[0].children[0].style.background);
            // event.target.style.visibility = 'hidden';

            if( event.target.parentNode.tagName.localeCompare('TD') == 0 && event.target.parentNode.innerText.localeCompare('') != 0){
                // console.log('Se arma');
                

                event.target.parentNode.parentNode.parentNode.rows[0].children[0].style.background = '';
                event.target.parentNode.parentNode.parentNode.rows[0].children[1].style.background = '';
                event.target.parentNode.parentNode.parentNode.rows[0].children[2].style.background = '';
                event.target.parentNode.parentNode.parentNode.rows[0].children[3].style.background = '';
                event.target.parentNode.parentNode.parentNode.rows[1].children[0].style.background = '';
                event.target.parentNode.parentNode.parentNode.rows[1].children[1].style.background = '';
                event.target.parentNode.parentNode.parentNode.rows[1].children[2].style.background = '';
                event.target.parentNode.parentNode.parentNode.rows[1].children[3].style.background = '';


                // console.log(event.target.parentNode.parentNode.parentNode.rows[0].children);
                
                if( event.target.parentNode.style.background.localeCompare('radial-gradient(circle, rgb(26, 33, 54) 0%, rgb(101, 101, 103) 82%)') == 0){
                    event.target.parentNode.style.background = '';
                }else{
                    // event.target.parentNode.style.backgroundColor = "rgb(101, 101, 103)";
                    // console.log('entro');
                    event.target.parentNode.style.background = 'radial-gradient(circle, rgba(26,33,54,1) 0%, rgba(101,101,103,1) 82%)'
                }

            }else{
                // console.log('No se arma');
            }

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
    /* background: rgb(28,29,33); */
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

#table-variables td{
    align-content: center;
    align-items: center;
    align-self: auto;
}

/* #table-variables td:checked{
    color: red;
    background-color: greenyellow;
} */

#table-variables div.celda-variable-nombre{
    height: 50%;
}
#table-variables div.celda-variable-lectura{
    display: inline-block;
    border-radius: 15px;
    background-color: #ffffff;
    color: #707070;
    width: 70%;
    align-self: auto;
}
#table-variables button{
    border-radius: 15px;
}
#table-variables button:empty {
   display: none;
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

/* #table-variables tr:first-child td:first-child{
    background: radial-gradient(circle, rgb(26, 33, 54) 0%, rgb(101, 101, 103) 82%);
} */

</style>