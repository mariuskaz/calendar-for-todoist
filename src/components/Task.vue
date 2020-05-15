<template>
    <div class="task-dialog">
        <i @click="status.taskId=0" class="material-icons big-icon square" style="position:absolute;right:10px;top:10px;font-size:22pt;;">close</i>
        <div>
            <div vv-if="project"><b>{{ project }}</b><br><br></div>
            <div style="float:left;height:80px;width:30px;font-weight:bold"><input type="checkbox" v-model="task.completed" @change="checkTask" style="display:inline-block;transform:scale(1.3);"></div>
            <div style="font-weight:normal;font-size:big"><a id="tasklink" target="blank" :href="task.url">{{ task.content }}</a></div>
            <div style="margin-top:15px;">
                <button class="info-label" :style="{ color: color }">{{ due }}</button>
                <button class="info-label">{{ task.user }}</button>
                <button v-if="hasTime" class="time-label" @click="setDuration" title="Click to change">{{ task.duration }}min</button>
            </div>
        </div>  
        <br><br>
        <span style="font-size:11pt;padding-left:7px">Comments&nbsp;&nbsp;<b>{{ commentsNo }}</b></span>
        <hr>
        <div class="empty" v-if="comments.length==0">No comments</div>
        <div class="notes">
            <div class="note" v-for="comment in comments" :key="comment.id">
                <p><b>{{ users[comment.postedUid].fullName }}</b></p>
                <div v-html="getNotes(comment.content)"/>
            </div>
        </div> 
    </div>
</template>

<script>
export default {
    props:['status','tasks','projects','notes','users'],
    name: 'task',

    computed: {

        task() {
            return this.tasks.filter(item => { return item.id == this.status.taskId })[0]
        },

        hasTime() {
            return this.task.due && this.task.due.string && this.task.due.string.indexOf(':') != -1
        },

        due() {
            return this.task.due == undefined ? "Not sheduled" : this.task.due.string
        },

        color() {
            if (this.task.due != undefined) {
                if ( new Date(this.task.due.date).getTime() < new Date().getTime() ) return "indianred"
            }
            return "green"
        },
        
        project() {
            if (this.projects[this.task.project_id]) return this.projects[this.task.project_id].name
            return "not found"
        },

        comments() {
            return this.notes.filter( note => { return note.itemId == this.status.taskId })
        },

        commentsNo() {
            if (this.comments.length == 0) return ""
            return this.comments.length
        }, 

    },

    methods: {

        getNotes(note) {
            if (note.substring(0,4)=='http' || note.substring(0,8)=='onenote:') return "<a target='_blank' href='"+note+"'>"+note.substring(0,70)+"</a>"
            return note
        },

        setDuration() {

            if (this.task.duration < 120) this.task.duration += 30
                else this.task.duration = 30

            let data = JSON.stringify({ duration: this.task.duration, completed: this.task.completed })
            localStorage.setItem(this.task.id, data)

        },

        openTask(url) {
            window.open(url)
        },

        checkTask() {
            let task = { id: this.task.id, completed: this.task.completed }
            this.$emit('updateTask', task)
        }
    }

}
</script>

<style>
.task-dialog {
    position:absolute;
    display:inline-block;
    background: rgba(250,250,250,1);
    border:1px solid silver;
    border-radius:4px;
    box-shadow: 0 24px 38px 3px rgba(0,0,0,0.14), 0 9px 46px 8px rgba(0,0,0,0.12), 0 11px 15px -7px rgba(0,0,0,0.2);
    min-height:240px;
    max-height:420px;
    width:360px;
    top:0;left:0;right:0;
    margin:5% auto;
    padding:50px;
    overflow:hidden;
    z-index:99999;
}

.footer {
    position:absolute;
    bottom:0;
    margin-bottom:20px;
}

.square {
    border-radius:4px;
    padding:4px;
}

.info-label {
    cursor:default;
    color:dimgray;
}

.time-label {
    color:green;
    min-width:30px;
    border:0;
    float:right;
}

.time-label:hover {
    color:green;
}

a, a:visited {
    color: black;
    text-decoration:none;
}

a:hover {
    text-decoration: underline;
}

.notes {
    overflow:hidden;
    overflow-y:auto;
    max-height:200px;
}

.notes:hover {
    oooverflow-y:scroll;
}

.note {
    color:black;
    font-size:11pt;
    padding-left:9px;
    whiteee-space:pre;
}

.note a {
    text-decoration: underline;
}

.empty {
    width:100%;
    padding:30px 0;
    color:lightgray;
    text-align:center;
}

</style>