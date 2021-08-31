<template>
    <div class="sidebar">
      
      <small>Darbuotojas:</small>
      <div class="section" style="margin-top:0px">
        <select v-model="status.person" @change="change" class="select">
          <option v-for='(person, id) in persons' :key='id' :value="id">{{ person.name }}</option>
        </select>
        </div>
        <button @click="status.connect=true" class="create" style="positionn:fixed;bottom:30px;margin:0 0 5px">Pridėti</button>
      
      <br><br>
      <small><a style="cursor:pointer" @click="sort" title="Perrikiuoti"><b>Užduotys</b></a><span style="float:right;borderr-radius:50px;background:lightgray;padding:4px 8px;color:gray;">{{ activeTasks.length }}</span></small>
      <div class="section" id="taskslist" style="padding:8px 0;height:calc(100vh - 245px);margin-top:5px;border-top:1px solid lightgray;width:100%">
        <div class="todo" draggable="true" @dragstart="drag" 
          :id="'t'+task.id" v-for="task in activeTasks" :key="task.id">
              <div style="float:left;height:30px;"><input v-model="task.completed" type="checkbox" @change="checkTask(task)" :style="{ borderColor: priority(task) }" style="display:inline-block"></div>
              <div style="float:left;width:155px;max-height:28px;overflow:hidden;" 
                @click="status.taskId = task.id"
                :class="{ completed: task.completed }"
                :title="task.content">{{task.content}}</div>
              <br>
              <div class="due" :style="{ color: color(task) }" style="float:left;font-size:8pt;color:red;padding-lefttt:22px;margin-top:-7px;">{{ due(task) }}</div>
        </div>
      </div>
    </div>
</template>

<script>
export default {
    props: ['status','persons', 'tasks'],
    name: 'sidebar',

    data() {
      return {
        order: 1
      }
    },

    computed: {

      activeTasks() {
        return this.tasks
        .filter(task => !task.completed )
        .sort((a, b) => { return this.order })
      }

    },

    methods: {

      sort() {
        this.order *= -1
      },

      change() {
        localStorage.setItem('user', this.status.person)
        this.$emit('sync', true)
      },

      due(task) {
        if (task.due) return task.due.string
        return "Not sheduled"
      },

      color(task) {
        if (task.due && new Date(task.due.date).getTime() < new Date().getTime() ) return 'indianred'
        return 'green'
      },

      drag(e) {
        console.log("drag", e.target.id)
        e.dataTransfer.setData("text", e.target.id);
      },

      checkTask(task) {
        let data = JSON.stringify({ duration: task.duration, completed: task.completed })
        localStorage.setItem(task.id, data)
      },

      priority(task) {
        let color = task.priority - 1,
        colors = [ "black", "#246fe0", "#eb8909", "#d1453b" ]
        return colors[color] || "black"
      },

    }
}
</script>

<style>

.sidebar {
    position:fixed;
    display:inline-block;;
    padding:10px;
    width:200px;
    top:55px;
}

.sidebar div {
  padding:5px 0;
}

.select {
  border:1px solid gray;
  padding:3px;
  width:90%;
}

.section {
  padding:10px;
  margin:10px 0;
  font-size: 10pt;
  overflow-x:hidden;
  overflow-y:hidden;
  scrollbar-width: thin;
}

.section:hover {
  overflow-y:auto;
}

.title {
  font-size:larger;
}

.todo {
    display:inline-block;
    cursor:pointer;
    font-size:small;
    overflow:hidden;
    line-height:16px;
    padding:0 1px;
    z-index:999;
}

.completed {
  text-decoration: line-through;
}

</style>