<script>
    export default {
        data() {
            return {
                selectProd: '',
            };
        },
        props: {
            mostrarPedidoModal: {
                default: false
            }
        },
        emits: ['update:mostrarPedidoModal'],
        watcher: {
            close() {
                this.$emit('update:mostrarPedidoModal', false);
            }
        },
        mounted(){
            this.getProdSistema();
        },
        methods:{
            getProdSistema(){
                axios({
                    method: 'get',
                    url: '/api/getProdSistema',
                    headers :{
                        Authorization: `Bearer ${localStorage.getItem('token')}`,
                        Accept :'application/json',
                    },
                }).then((response) => {
                    if((response.data.data).length > 0){
                        let dataProd = '<option value="" selected>Seleccione un opción</option>';
                        $.each((response.data.data), function(key, value){
                            dataProd += `<option value="${value.id}" data-iva="${value.producto_iva}" data-valor="${value.producto_valor}">${value.producto_nombre}</option>`;
                        })
                        this.selectProd = dataProd;
                    }
                });
            },
            setPediData(){
                let vali = 0;
                $('.valiPedi').each(function(){
                    if($(this).val() == "" || $(this).val() <= 0){
                        $(this).removeClass( "border-teal-100" ).addClass( "border-red-500" );
                        Swal.fire({
                            title: "¡UN MOMENTO!",
                            text: `Debe registrar ${$(this).data('titulo')} o validar la inforamción registrada.`,
                            icon: "error",
                            showCancelButton: false,
                            focusConfirm: false,
                            confirmButtonText: 'Aceptar',
                        });
                        vali = 1;
                        return false;
                    }else{
                        $(this).removeClass( "border-red-500" ).addClass( "border-teal-100" );
                        vali = 0;
                    }
                })

                if(vali == 0){
                    let arrProd = {
                        'id' : $("#pediProdCrea").val(),
                        'iva' : $("#pediProdCrea").find(':selected').attr('data-iva'),
                        'valor' : $("#pediProdCrea").find(':selected').attr('data-valor'),
                    }
                    axios({
                        method: 'post',
                        url: '/api/setPediData',
                        headers :{
                            Authorization: `Bearer ${localStorage.getItem('token')}`,
                            Accept :'application/json',
                        },
                        data: {
                            prod: arrProd,
                            cantidad: $("#pediCantCrea").val(),
                        }
                    }).then((response) => {
                        if(response.data.estado== "OK"){
                            Swal.fire({
                                title: "PERFECTO",
                                text: 'Se registor el pedido con éxito.',
                                icon: "success",
                                confirmButtonText: "Aceptar",
                            }).then((result) => {
                                if (result.isConfirmed) {
                                    location.reload();
                                }
                            });
                        }else{
                            Swal.fire({
                                title: "ERROR",
                                text: 'Ocurrió un problema al registrar el pedido.',
                                icon: "errpr",
                                confirmButtonText: "Aceptar",
                            })
                        }
                    }, (error) => {
                        Swal.fire({
                            title: "ERROR",
                            text: 'Ocurrió un problema al registrar el producto.',
                            icon: "errpr",
                            confirmButtonText: "Aceptar",
                        })
                    });
                }
            }
        }
    }
</script>

<template>
    <div v-if="mostrarPedidoModal" class="modal-overlay col-span-12" @click="close">
        <div class="modal-content" @click.stop>
            <button class="close-button mb-5" @click="close">X</button>
            <div class="grid grid-cols-12">
                <div class="col-span-12 mb-5">
                    <p class="text-md font-medium">Crear pedido</p>
                </div>
                <div class="col-span-12">
                    <div class="mb-8">
                        <label for="pediProdCrea" class="block text-gray-700 text-sm font-bold mb-2">
                            <span class="text-red-500">&nbsp;*</span>
                            Seleccione un producto
                        </label>
                        <div class="mt-1 relative rounded-md shadow-sm">
                            <select v-html="selectProd" id="pediProdCrea" class="block pr-10 shadow appearance-none border-2 border-teal-100 rounded w-full py-2 px-4 text-gray-700 mb-3 leading-tight focus:outline-none focus:bg-white focus:border-teal-500 transition duration-500 ease-in-out valiPedi">

                            </select>
                        </div>
                    </div>
                </div>
                <div class="col-span-12">
                    <div class="mb-8">
                        <label for="pediCantCrea" class="block text-gray-700 text-sm font-bold mb-2">
                            <span class="text-red-500">&nbsp;*</span>
                            Ingrese la cantidad de producto deseado
                        </label>
                        <div class="mt-1 relative rounded-md shadow-sm">
                            <input type="number" id="pediCantCrea" class="block pr-10 shadow appearance-none border-2 border-teal-100 rounded w-full py-2 px-4 text-gray-700 mb-3 leading-tight focus:outline-none focus:bg-white focus:border-teal-500 transition duration-500 ease-in-out valiPedi" data-titulo="la cantidad de producto" />
                        </div>
                    </div>
                </div>
                <div class="col-span-12">
                    <div class="mb-4 text-center">
                        <button class="transition duration-500 bg-teal-500 hover:bg-teal-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" type="button" @click="setPediData()">
                            Ingresar
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
