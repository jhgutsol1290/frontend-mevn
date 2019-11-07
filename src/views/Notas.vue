<template>
    <div class="container">
        <h1>Notas</h1>
        <b-alert
            :show="dismissCountDown"
            dismissible
            :variant="mensaje.color"
            @dismissed="dismissCountDown=0"
            @dismiss-count-down="countDownChanged"
        >
            {{mensaje.texto}}
        </b-alert>

        <form @submit.prevent="editarNota(notaEditar)" v-if="editar">   
            <h3 class="text-center">Editar Nota</h3>
            <input type="text" placeholder="Ingrese un Nombre" class="form-control my-2" v-model="notaEditar.nombre">
            <input type="text" placeholder="Ingrese Descripción" class="form-control my-2" v-model="notaEditar.descripcion">
            <b-button class="btn btn-warning m-2" type="submit">Confirmar</b-button>
            <b-button class="btn btn-secondary m-2" type="submit" @click="editar = false, nota.nombre = '', nota.descripcion = ''">Cancelar</b-button>
        </form>

        <form @submit.prevent="agregarNota()" v-if="!editar">   
            <h3 class="text-center">Agregar nueva Nota</h3>
            <input type="text" placeholder="Ingrese un Nombre" class="form-control my-2" v-model="nota.nombre">
            <input type="text" placeholder="Ingrese Descripción" class="form-control my-2" v-model="nota.descripcion">
            <b-button class="btn btn-success btn-block my-2" type="submit">Agregar</b-button>
        </form>

        <table class="table">
            <thead>
                <tr>
                    <th scope="col">#</th>
                    <th scope="col">Nombre</th>
                    <th scope="col">Descripción</th>
                    <th scope="col">Fecha</th>
                    <th scope="col">Acciones</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in notas" :key="index">
                    <th scope="row">{{item._id}}</th>
                        <td>{{item.nombre}}</td>
                        <td>{{item.descripcion}}</td>
                        <td>{{item.date}}</td>
                        <td>
                            <b-button 
                                class="btn-warning btn-sm mx-2" 
                                @click="activarEdicion(item._id)"
                            >
                                Actualizar
                            </b-button>
                            <b-button 
                                class="btn-danger btn-sm mx-2" 
                                @click="eliminarNota(item._id)"
                            >
                                Eliminar
                            </b-button>
                            <!-- <b-button @click="alerta()">Accion</b-button> -->
                        </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>

export default {
    data() {
        return {
            notas: [],
            dismissSecs: 5,
            dismissCountDown: 0,
            mensaje: {
                color: '',
                texto: ''
            },
            nota: {
                nombre: '',
                descripcion: ''
            },
            editar: false,
            notaEditar: {}
        }
    },
    created() {
        this.listarNotas()
    },
    methods: {
        alerta() {
            this.mensaje.color = 'success'
            this.mensaje.texto = 'Probando'
            this.showAlert()
        },
        listarNotas() {
            this.axios.get('/nota')
                .then(res => {
                    this.notas = res.data
                })
                .catch(e => {
                    console.log(e.response)
                })
        },
        agregarNota() {
            this.axios.post('/nueva-nota', this.nota)
                .then(res => {
                    // this.notas.push(res.data)
                    this.listarNotas()
                    this.nota.nombre = ''
                    this.nota.descripcion = ''
                    this.mensaje.color = 'success'
                    this.mensaje.texto = 'Nota agregada satisfactoriamente'
                    this.showAlert()
                })
                .catch(e => {
                    console.log(e.response)
                    if(e.response.data.error.errors.nombre.message) {
                        this.mensaje.texto = e.response.data.error.errors.nombre.message
                    } else {
                        this.mensaje.texto = 'Error del sistema'
                    }
                    this.mensaje.color = 'danger'
                    this.showAlert()
                })
        },
        editarNota(notaEditar) {
            this.axios.put(`/nota/${notaEditar._id}`, notaEditar)
                .then(res => {
                    this.listarNotas()
                    this.nota.nombre = ''
                    this.nota.descripcion = ''
                    this.mensaje.color = 'success'
                    this.mensaje.texto = 'Nota actualizada satisfactoriamente'
                    this.showAlert()
                })
                .catch(e => {
                    console.log(e)
                })
                this.agregar = true
        },
        eliminarNota(id) {
            this.axios.delete(`/nota/${id}`)
                .then(res => {
                    // const index = this.notas.findIndex(item => item._id === res.data._id)
                    // this.notas.splice(index, 1)
                    this.listarNotas()
                    this.mensaje.texto = 'Nota eliminada'
                    this.mensaje.color = 'danger'
                    this.showAlert()
                })
                .catch(e => {
                    console.log(e.response)
                })
        },
        activarEdicion(id) {
            this.editar = true
            console.log(id)
            this.axios.get(`/nota/${id}`)
                .then(res => {
                    this.notaEditar = res.data
                })
                .catch(e => {
                    console.log(e.response)
                })
        },
        countDownChanged(dismissCountDown) {
            this.dismissCountDown = dismissCountDown
        },
        showAlert() {
            this.dismissCountDown = this.dismissSecs
        }
    }
}


</script>