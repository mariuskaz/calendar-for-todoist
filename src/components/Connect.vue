<template>
    <div class="dialog">
        <label>Vartotojas</label><br>
        <input v-model="input.user" id='user' type="text" style="width:130px" />
        <br><br>
        <label>Todoist raktas</label><br>
        <input v-model="input.token" type="text" style="width:300px"/>
        <div class="footer">
            <button @click="connect" class="create">ĮTRAUKTI</button>
            <button @click="status.connect=false">BAIGTI</button>
        </div>
    </div>
</template>

<script>
export default {
    props:['status','input'],
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

</style>