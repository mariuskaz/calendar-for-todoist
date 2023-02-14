<template>
    <div class="dialog">
        <i @click="status.connect=false" class="material-icons big-icon square" style="position:absolute;right:10px;top:10px;font-size:22pt;;">close</i>
        <label>{{ translate.user }}</label><br>
        <input v-model="input.user" id='user' type="text" style="width:130px" />
        <input v-model="input.color" id='color' type="color" style="width:26px;vertical-align:bottom;" />
        <br><br><br>
        <label>{{ translate.token }}</label><br>
        <input v-model="input.token" type="text" style="width:300px"/>
        <div class="footer">
            <button @click="connect" class="create">{{ translate.connect }}</button>
            <button @click="status.connect=false" class="cancel">{{ translate.close }}</button>
        </div>
    </div>
</template>

<script>
export default {
    props:['status','input','translate'],
    name: 'connect',

    mounted() {
        document.getElementById('user').focus()
    },

    methods: {
        connect() {
            this.$emit('add')
            this.status.connect=false
        }
    }
}
</script>

<style>
.dialog {
    position:absolute;
    display:inline-block;
    background: rgba(250,250,250,1);
    border:1px solid silver;
    border-radius:4px;
    box-shadow: 0 24px 38px 3px rgba(0,0,0,0.14), 0 9px 46px 8px rgba(0,0,0,0.12), 0 11px 15px -7px rgba(0,0,0,0.2);
    height:170px;
    width:300px;
    top:0;left:0;right:0;
    margin:150px auto;
    padding:40px 50px 50px;
    z-index:99999;
}

.footer {
    position:absolute;
    bottom:30px;
    padding-bottom:20px;
}

.cancel {
    text-transform: uppercase;
}

</style>