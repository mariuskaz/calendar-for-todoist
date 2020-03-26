<template>
    <transition name='fade'>
    <div class="container" :key="status.view">
      <div class="grid-container" style="width:calc(100% - 20px);">
        <div class="grid-header"></div>
        <div class="grid-header"
            @click="newTask(d, 0, $event)"
            @dragover="allowDrop"
            @drop="drop(d, 0, $event)" 
            @touchend="touchend"

            :class="{ today: isToday(day) }"
            :key='d' 
            v-for='(day, d) in days'>

            <div style="padding-left:10px">
                {{ getDay(day) }}
                <div style="margin-bottom:20px;font-size:10pt;">
                    {{ getWeekDay(day) }}
                </div>
            </div>
            
            <!-- ALL DAY TASKS -->
            <div class='task' draggable="true"
                @click.stop="status.taskId = task.id" 
                @dragstart="drag"
                @touchstart="touchstart"
                @touchmove="touchmove"

                :id="task.id" 
                :title="task.content" 
                :class="{ fullhour: false, completed: task.completed }"
                :style="{ background: setColor(task) }"
                :key="i" v-for="(task, i) in allDayTasks(d)">

                    {{ task.content }}

            </div>

        </div>
      </div>

      <div class="grid-container grid-view">

        <div class="grid-column" style="font-size:8pt">
            <div class="grid-item">07:00</div>
            <div class="grid-item">08:00</div>
            <div class="grid-item">09:00</div>
            <div class="grid-item">10:00</div>
            <div class="grid-item">11:00</div>
            <div class="grid-item">12:00</div>
            <div class="grid-item">13:00</div>
            <div class="grid-item">14:00</div>
            <div class="grid-item">15:00</div>
            <div class="grid-item">16:00</div>
            <div class="grid-item">17:00</div>
            <div class="grid-item">18:00</div>
            <div class="grid-item">19:00</div>
            <div class="grid-item">20:00</div>
            <div class="grid-item">21:00</div>
            <div class="grid-item">22:00</div>
            <div class="grid-item">23:00</div>
        </div>

        <div class="grid-column" 
            v-for="(day, d) in days" :key="d">
            
            <div class="grid-item" 
                @click="newTask(d, h, $event)"
                @dragover="allowDrop"
                @drop="drop(d, h, $event)" 
                @touchend="touchend"

                :date="days[d]"
                :hour="h"
                :key="h" v-for="h in hours">

                    <!-- HOUR TASKS -->
                    <div class='task'  
                        @click.stop="status.taskId = task.id" 
                        @dragstart="drag"
                        @touchstart="touchstart"
                        @touchmove="touchmove"
                        
                        :draggable="draggable"
                        :class="{ 
                            completed: task.completed,
                            overflow: isOverflow(d, h, task),
                        }"

                        :style="{ 
                            background: setColor(task),
                            position: 'absolute',
                            height: setHeight(task),
                            marginTop: setPosition(task),
                        }" 

                        :id="task.id" 
                        :title="task.content" 
                        :key="i" v-for="(task, i) in getHourTasks(d, h)">

                            {{ task.content }}

                    </div>

                    <ruler :style="{ marginTop: getRulerPos() }" 
                        v-if="!status.busy && isNow(day, h)" />

            </div>

        </div>
        
      </div>

        <todo v-if="status.new_task" 
            @add="addTask" :status="status" :due="due"/>
        <loading v-if="status.busy" />

    </div>
    </transition>
</template>

<script>
import ruler from './Ruler.vue'
import todo from './QuickTask.vue'
import loading from './Loading.vue'
export default {
    props: ['days', 'tasks', 'status'],
    name: 'calendar',
    components: {
        todo,
        ruler,
        loading,
    },
    data() {
        return {
            hours: 17,
            weekdays: [
                'sekmadienis', 'pirmadienis', 'antradienis', 'trečiadienis', 'ketvirtadienis', 'penktadienis', 'šeštadienis'
            ],
            draggable: true,
            point: 0,
            due: "today",
            touch: {
                activeTask: 0,
                startdrag: new Function,
                start:false,
                moved:false,
                startX:-1,
                startY:-1,
                x: 0,
                y: 0
            }
        }
    },

    methods: {

        /* put overflow check to App->getTasks() */
        isOverflow(day, hour, todo) {
            hour += 6
            let current = new Date(todo.due.date),

            same = this.tasks
            .filter (task => { return task.due != undefined} )
            .filter (task => { return task.due.string != undefined} )
            .filter (task => { return task.due.string.indexOf(":") > 0} )
            .filter( task => { 
                let date = new Date(task.due.date),
                duration = new Date(date.getTime() + (task.duration * 60 * 1000)) //miliseconds
                return duration > current
            })
            .sort((a, b) => {
                if (a.due && b.due) {
                    return new Date(a.due.date) - new Date(b.due.date) > 0 ? 1 : -1
                } else if (a.due) {
                    return -1
                }
                return 1
            })

            if (same.length >= 1) {
                if (same[0].id != todo.id) return true
            } 

            return false
        },

        setHeight(task) {
            return task.duration * 26 / 30 - 1  + "px"
        },

        setPosition(task) {
            let pos = Math.floor(new Date(task.due.date).getMinutes()*50/60)
            if (pos > 0) pos += 1
            return pos + 'px'
        },

        allDayTasks(day) {
            return this.tasks
            .filter (task => { return task.due != undefined} )
            .filter (task => { return task.due.string != undefined} )
            .filter (task => { return task.due.string.indexOf(":") == -1} )
            .filter( task => { return new Date(task.due.date).toLocaleDateString() == this.days[day]})
        },

        getHourTasks(day, hour) {
            hour += 6
            return this.tasks
            .filter (task => { return task.due != undefined} )
            .filter (task => { return task.due.string != undefined} )
            .filter (task => { return task.due.string.indexOf(":") > 0} )
            .filter( task => { return new Date(task.due.date).toLocaleDateString() == this.days[day]})
            .filter( task => { return new Date(task.due.date).getHours() == hour})
            .sort( (a, b) => { return b.duration - a.duration })
        },

        getDay(dateString) {
            return new Date(dateString).getDate()
        },

        getWeekDay(dateString) {
            return this.weekdays[new Date(dateString).getDay()]
        },        

        isToday(d) {
            return new Date(d).toLocaleDateString() == new Date().toLocaleDateString()
        },

        isNow(day, hour) {
            return new Date(day).toLocaleDateString() == new Date().toLocaleDateString() &&
                   hour + 6 == new Date().getHours() 
        },

        isOverdue(date, hour) {
            return new Date(date).toLocaleDateString() < new Date().toLocaleDateString()
        },

        getRulerPos() {
            return Math.floor(new Date().getMinutes()*50/60 - 0) + 'px'
        },

        setColor(task){
            
            if (!task.completed) return this.status.color

            // strip the leading # if it's there
            let hex = this.status.color.replace(/^\s*#|\s*$/g, ''),
            percent = 60

            // convert 3 char codes --> 6, e.g. `E0F` --> `EE00FF`
            if(hex.length == 3){
                hex = hex.replace(/(.)/g, '$1$1')
            }

            let r = parseInt(hex.substr(0, 2), 16),
                g = parseInt(hex.substr(2, 2), 16),
                b = parseInt(hex.substr(4, 2), 16)

            return '#' +
            ((0|(1<<8) + r + (256 - r) * percent / 100).toString(16)).substr(1) +
            ((0|(1<<8) + g + (256 - g) * percent / 100).toString(16)).substr(1) +
            ((0|(1<<8) + b + (256 - b) * percent / 100).toString(16)).substr(1);

        },

        allowDrop(e) {
            e.preventDefault()
        },

        drag(e) {
            console.log('drag', e.target.id)
            e.dataTransfer.setData("text", e.target.id);
        },

        drop(d, h, event) {
            event.preventDefault()
            let id = event.dataTransfer.getData("text"),
            hour = new String("0" + (h + 6)).substr(-2),
            due = { string: "" }
            if (id.substr(0,1) == "t") id = id.substr(1)
            this.tasks.forEach( task => {
                if (task.id == id) {
                    if (task.due == undefined) task.due = {}
                    if ( h > 0 ) {
                        let rect = event.target.getBoundingClientRect(),
                        clientY = event.clientY - rect.top,
                        mins = "00"

                        if (clientY > 29) mins = "30"

                        task.due.date = this.days[d] + "T" + hour + ":" + mins + ":00"
                        task.due.string = this.days[d] + " " + hour + ":" + mins
                    } else {
                        task.due.date = this.days[d] 
                        task.due.string = this.days[d]
                    }
                    due = task.due
                }
            })
            console.log('drop', id, due.string)
            this.$emit('dropTask', id, due.string)
        },

        touchstart (e) {
            let touch = this.touch
            this.touch.startdrag = setTimeout( function() {
                if (!touch.moved) {
                    touch.activeTask = e.target.id
                    touch.startX = e.touches[0].clientX
                    touch.startY = e.touches[0].clientY
                    touch.start = true
                    e.target.classList.add('selected')
                    console.log("touch start")
                }
            }, 1000)
        },

        touchmove(e) {
            this.touch.moved = true
            clearTimeout(this.touch.startdrag)
            if (this.touch.start) {
                e.preventDefault()
                this.touch.x = e.touches[0].clientX
                this.touch.y = e.touches[0].clientY
                //e.target.style.position = "absolute"
               // e.target.style.top = this.touch.y + "px"
            }
        },

        touchend(e) {
            if (this.touch.start && this.touch.moved) {
                let due = { string: "" },
                target = document.elementFromPoint(this.touch.x, this.touch.y)
                console.log("touchdrag")
                if (target != undefined) {
                    let date = target.getAttribute('date'),
                    h = parseInt(target.getAttribute('hour')),
                    hour = new String("0" + (h + 6)).substr(-2)
                    this.tasks.forEach( task => {
                        if (task.id == this.touch.activeTask) {
                            if (task.due == undefined) task.due = { date:"", string:"" }
                            if ( h > 0 ) {
                                task.due.date = date + "T" + hour +":00:00"
                                task.due.string = date + " " + hour +":00"
                            } else {
                                task.due.date = date
                                task.due.string = date
                            }
                            due = task.due
                        }
                    })
                }
            }
            this.touch.moved = false
            this.touch.start = false
            clearTimeout(this.touch.startdrag)
        },

        newTask(d, h, e) {
            
            let rect = e.target.getBoundingClientRect(),
            clientY = e.clientY - rect.top,
            mins = "00"

            if (clientY > 29) mins = "30"

            if (h == 0) {
                this.due = this.days[d]
            } else {
                let hour = new String("0" + (h + 6)).substr(-2)
                this.due = this.days[d] + " " + hour + ":" + mins
            }
            this.status.new_task = true

        },

        addTask(todo) {
            let task = {
                content: todo,
                due_string: this.due
            }
            this.$emit('addTask', task)
        }


    }
}
</script>

<style>
.container {
    display:inline-block;
    width:77%; 
    margin:20px 0 0 240px;
    font-size:0.9em;

    display: flex;
    flex-flow: column;
    height: calc(100vh - 90px);
}

.grid-container{
    display: grid;
    grid-template-columns: 5% 19% 19% 19% 19% 19%;
    border-bottom:1px solid lightgray
}

.grid-view {
    overflow-y:auto;
    overflow-x:hidden;
    scrollbar-width: thin;
    flex-grow : 1;
}

.grid-column {
    border-right:1px solid lightgray;
}

.grid-header {
    color: #70757a;
    font-size:18pt;
    min-height:60px;
}

.day-name { 
    font-size:10pt;
    padding:0px 0;
}

.grid-item {
    border-bottom:1px solid lightgray;
    height:50px;
    padding-bottom:2px;
    user-select:none;
    position:relative;
}

.task {
    float:left;
    border-radius:4px;
    cursor:pointer;
    font-size:9pt;
    overflow:hidden;
    height:25px;
    line-height:18px;
    min-height:25px;
    padding:0 2px;
    width:84%;
    margin-bottom:1px;
    z-index:999;
    resize:none; /* vertical */

    /* border-bottom:1px solid white; */
    color:white;
    font-size:9pt;
    line-height:18pt;
    padding-left: 4px;
    text-decoration:none;
    overflow:hidden;
}

.completed {
    text-decoration: line-through;
    filter: brightness(1);
    color:dimgray;
}

.overflow {
    border:1px solid white;
    margin-left:40%;
    width:50%;
    z-index:9999;
}

.selected {
    opacity:0.5;
}

.today {
    color:red;
}

.fade-enter-active /*, .fade-leave-active */ {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0
}

</style>