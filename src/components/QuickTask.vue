<template>
    <div class="todo-dialog">
        <i @click="status.new_task = false" class="material-icons big-icon square" style="position:absolute;right:10px;top:10px;font-size:22pt;;">close</i>
        Nauja užduotis<br>
        <input id="todo" v-model="todo" type="text" style="width:300px;margin-top:5px"
            @keyup.enter="add" @keyup.escape="status.new_task = false"/>
        <div class="todo-footer">
            <button @click="add" class="create">PRIDĖTI</button>
            <!-- button @click="status.new_task = false">BAIGTI</button -->

            <button style="float:right">{{ due }}</button>
        </div>
    </div>
</template>

<script>
export default {
    props:['status', 'due'],
    name: 'todo',
    data() {
        return {
            todo: ""
        }
    },
    mounted() {
        document.getElementById('todo').focus()
    },
    methods: {
        add() {
            this.$emit('add', this.todo)
            this.status.new_task = false
        }
    }
}
</script>

<style>
.todo-dialog {
    position:absolute;
    display:inline-block;
    background: rgba(250,250,250,1);
    border:1px solid silver;
    border-radius:4px;
    box-shadow: 0 24px 38px 3px rgba(0,0,0,0.14), 0 9px 46px 8px rgba(0,0,0,0.12), 0 11px 15px -7px rgba(0,0,0,0.2);
    height:90px;
    width:300px;
    top:0;left:0;right:0;
    margin:150px auto;
    padding:30px;
    z-index:99999;
}

.todo-footer {
    position:absolute;
    bottom:25px;
    left:30px; right:30px;
}

</style>