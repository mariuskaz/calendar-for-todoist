<template>
	<div id="app">
		<heading :days='days' :status='status' @today='today' @back='back' @forward='forward' @sync="sync" />
		<sidebar :persons='persons' :status='status' :tasks="tasks" @sync='sync' />
		<calendar :tasks='tasks' :days='days' :status='status' @addTask="addTask" @dropTask="dropTask" />
		<connect v-if="status.connect" :status="status" :input='input' @add='add' />
		<task v-if="status.taskId > 0" :status="status" :tasks='tasks' :details="details" :projects='projects' :notes='notes' :users='users' @updateTask="updateTask"/>
	</div>
</template>

<script>
import heading from './components/Heading.vue'
import sidebar from './components/Sidebar.vue'
import calendar from './components/Calendar.vue'
import connect from './components/Connect.vue'
import task from './components/Task.vue'

var Todoist = require('exports-loader?Todoist!./todoist.js')

export default {
	name: 'app',
	components: {
		heading,
		sidebar,
		connect,
		calendar,
		task,
	},

	data() {
		return {
			status: {
				active: true,
				view: 1,
				person: 0,
				connect: false,
				userId: 0,
				userName: '',
				color: 'gray',
				taskId: 0,
				inboxId: 0,
				busy: true,
				new_task: false,
				sync: true,
								lastSync:'pending'
			},

			input: {
				user: '',
				token: '',
				color:'#008B8B',
			},

			persons: [
				{ name: 'Pasirinkite...' },
			],

			days: [],
			tasks: [],
			details: {},
			projects: {},
			notes: [],
			users: {},

		}
	},

	mounted() {
		this.today()

		if (localStorage['persons']) {
			this.persons = JSON.parse(localStorage['persons'])
			this.status.person = localStorage.getItem('user') || 0
		}

		if (this.status.person > 0)  this.sync()
			else this.status.busy = false
		
		let status = this.status,
		sync = this.sync

		document.onvisibilitychange = function() { 
			status.active = !status.active
			if (status.active) sync()
		}

	},

	methods: {

		add() {

			this.persons.push({ name: this.input.user, token: this.input.token, color: this.input.color  })
			this.status.person = this.persons.length - 1
			localStorage.setItem('persons', JSON.stringify(this.persons))
			this.sync()
			this.input.user = ''
			this.input.token = ''
			
		},

		back(d = 1) {

			this.days = this.days.map(day => {
					let date = new Date(day)
					date.setDate(date.getDate() - d)
					return date.toLocaleDateString()
			})
			if (d > 1) this.status.view += 1

		},

		forward(d = 1) {

			this.days = this.days.map(day => {
					let date = new Date(day)
					date.setDate(date.getDate() + d)
					return date.toLocaleDateString()
			})
			if (d > 1) this.status.view += 1

		},

		sync(wipe = false) {

			if (wipe) this.tasks = []
			if (this.status.person == 0) return
			
			let id = this.status.person,
			todoist = new Todoist(),
			update = this.update
			this.status.busy = true
			todoist.token = this.persons[id].token || 'none'
			todoist.sync.oncomplete = function(data) {
				console.log('sync...') 
				let user = 'user' + data.user.id,
				items = localStorage[user] ? JSON.parse(localStorage[user]) : {}
				for (let item in items) items[item].completed = true
				data.items = { ...items, ...data.items }
				update(data, wipe)
				localStorage.setItem(user, JSON.stringify(data.items))
			}

			this.status.sync = true
			todoist.sync()

		},

		update(data, permanent = false) {

			this.status.userId = data.user.id
			this.status.userName = data.user.fullName
			this.status.color = this.persons[this.status.person].color

			this.projects = {...data.projects}
			this.users = {...data.collaborators}
			this.tasks = []
			this.notes = []

      		for (let item in data.notes) 
				this.notes.push(data.notes[item])

			for (let id in this.projects) {
				let project = this.projects[id].name
				if (project == 'Inbox') 
					this.status.inboxId = id
			}

			if (permanent) 
				document.getElementById('taskslist').scrollTop = 0
			
			for (let id in data.items) {
				let item = data.items[id],
				duration = 30

				if (localStorage[item.id]) {
					let data = JSON.parse(localStorage[item.id])
					duration = data.duration || 30
				}

				if (item.responsibleUid == this.status.userId || item.projectId == this.status.inboxId) {
					let desc = item.content
					if (desc.indexOf('[') == 0 && desc.indexOf('](' != -1)) desc = desc.split('](')[0].substring(1)
					this.tasks.push({ 
						id: item.id, 
						user: this.status.userName,
						project_id: item.projectId,
						completed: item.completed,
						content: desc, 
						due: item.due,
						url: "https://todoist.com/showTask?id=" + item.id,
						priority: item.priority,
						duration: duration,
						overflow: false
					})
				}
			}

			this.tasks.sort((a, b) => {
				if (a.due && b.due) {
					let delta = new Date(a.due.date) - new Date(b.due.date)
					return delta > 0 ? 1 : -1
				} else if (a.due) {
					return -1
				}
				return 1
			})

			console.log('tasks', this.tasks.length)
			this.status.busy = false

			this.status.lastSync = new Date().toLocaleTimeString()
			this.status.sync = false
		},

		getDay(day = 0) {

			let date = new Date
			return new Date(date.setDate(date.getDate() + day)).toLocaleDateString()

		},

		today() {

			let date = new Date,
			day = date.getDay()
			this.days = [
				this.getDay(1 - day),
				this.getDay(2 - day),
				this.getDay(3 - day),
				this.getDay(4 - day),
				this.getDay(5 - day),
			]
			this.status.view += 1

		},

		addTask(task) {
			let id = this.status.person
			if (id == 0 || this.persons[id].token.length == 0) return

			let headers = {
					'Authorization': 'Bearer ' + this.persons[id].token,
					'Content-Type': 'application/json'
			}

			fetch('https://api.todoist.com/rest/v1/tasks', { 
					method: 'POST',
					headers : headers,
					body: JSON.stringify(task)
			})

			.then(response => {
					this.sync()
			})


		},

		updateTask(task) {
			let id = this.status.person,
			user = 'user'+this.status.userId

			if (id == 0 || this.persons[id].token.length == 0) return

			let headers = {
					'Authorization': 'Bearer ' + this.persons[id].token,
			},

			action = task.completed ? 'close' : 'reopen'

			fetch('https://api.todoist.com/rest/v1/tasks/' + task.id + '/' + action, { 
					method: 'POST',
					headers : headers
			})

			.then(response => {
					if (!response.ok) console.error('task not updated:', response.status)
					let items = localStorage[user] ? JSON.parse(localStorage[user]) : {}
					if (items[task.id] == undefined) items[task.id] = {}
					items[task.id].completed = task.completed
					localStorage.setItem(user, JSON.stringify(items) )
			})

		},

		dropTask(taskId, date) {
			let id = this.status.person
			if (id == 0 || this.persons[id].token.length == 0) return

			console.log('change', date)

			let headers = {
					'Authorization': 'Bearer ' + this.persons[id].token,
					'Content-Type': 'application/json'
			},

			data = {
				due_string: date
			}

			fetch('https://api.todoist.com/rest/v1/tasks/' + taskId, { 
					method: 'POST',
					headers : headers,
					body: JSON.stringify(data)
			})

			.then(response => {
					this.sync()
			})
		},

		sortTasks() {
			this.tasks.sort((a, b) => { return -1 })
		}

	}
}
</script>

<style>
body {
	height:100vh;
	overflow:hidden;
}

button {
	cursor: pointer;
}

div {
	padding:0px;
}

#app {
	font-family: 'Avenir', Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	color: #2c3e50;
	margin-top: 60px;
}


/* BUTTONS */

.button,
button,
input[type="button"],
input[type="image"],
input[type="reset"],
input[type="submit"] {
	background: -moz-linear-gradient(top, #f5f5f5, #f1f1f1);
	background: -ms-linear-gradient(top, #f5f5f5, #f1f1f1);
	background: -o-linear-gradient(top, #f5f5f5, #f1f1f1);
	background: -webkit-linear-gradient(top, #f5f5f5, #f1f1f1);
	background: linear-gradient(top, #f5f5f5, #f1f1f1);
	border: 1px solid #dcdcdc;
	-moz-border-radius: 2px;
	-webkit-border-radius: 2px;
	border-radius: 2px;
	-moz-box-shadow: none;
	-webkit-box-shadow: none;
	box-shadow: none;
	color: #333;
	cursor: pointer;
	font-family: arial, sans-serif;
	font-size: 11px;
	font-weight: bold;
	height: 29px;
	line-height: 27px;
	margin: 0;
	min-width: 72px;
	outline: 0;
	padding: 0 8px;
	text-align: center;
	white-space: nowrap;
}

.button {
	box-sizing: border-box;
	display: inline-block;
	padding: 0;
}

.button.action,
.button.blue,
button.action,
button.blue,
input[type="button"].action,
input[type="button"].blue,
input[type="submit"].action,
input[type="submit"].blue {
	background: -moz-linear-gradient(top, #4d90fe, #4787ed);
	background: -ms-linear-gradient(top, #4d90fe, #4787ed);
	background: -o-linear-gradient(top, #4d90fe, #4787ed);
	background: -webkit-linear-gradient(top, #4d90fe, #4787ed);
	background: linear-gradient(top, #4d90fe, #4787ed);
	border: 1px solid #3079ed;
	color: #fff;
}

.button.create,
.button.red,
button.create,
button.red,
input[type="button"].create,
input[type="button"].red,
input[type="submit"].create,
input[type="submit"].red {
	background: -moz-linear-gradient(top, #dd4b39, #d14836);
	background: -ms-linear-gradient(top, #dd4b39, #d14836);
	background: -o-linear-gradient(top, #dd4b39, #d14836);
	background: -webkit-linear-gradient(top, #dd4b39, #d14836);
	background: linear-gradient(top, #dd4b39, #d14836);
	border: 1px solid transparent;
	color: #fff;
	text-shadow: 0 1px rgba(0, 0, 0, .1);
	text-transform: uppercase;
}

.button.green,
.button.share,
button.green,
button.share,
input[type="button"].green,
input[type="button"].share,
input[type="submit"].green,
input[type="submit"].share {
	background: -moz-linear-gradient(top, #3d9400, #398a00);
	background: -ms-linear-gradient(top, #3d9400, #398a00);
	background: -o-linear-gradient(top, #3d9400, #398a00);
	background: -webkit-linear-gradient(top, #3d9400, #398a00);
	background: linear-gradient(top, #3d9400, #398a00);
	border: 1px solid #29691d;
	color: #fff;
	text-shadow: 0 1px rgba(0, 0, 0, .1);
}

.button:hover,
button:hover,
input[type="button"]:hover,
input[type="image"]:hover,
input[type="reset"]:hover,
input[type="submit"]:hover {
	background: -moz-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: -ms-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: -o-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: -webkit-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: linear-gradient(top, #f8f8f8, #f1f1f1);
	border: 1px solid #c6c6c6;
	color: #111;
	text-decoration: none;
}

.button.action:hover,
.button.blue:hover,
button.action:hover,
button.blue:hover,
input[type="button"].action:hover,
input[type="button"].blue:hover,
input[type="submit"].action:hover,
input[type="submit"].blue:hover {
	background: -moz-linear-gradient(top, #4d90fe, #357ae8);
	background: -ms-linear-gradient(top, #4d90fe, #357ae8);
	background: -o-linear-gradient(top, #4d90fe, #357ae8);
	background: -webkit-linear-gradient(top, #4d90fe, #357ae8);
	background: linear-gradient(top, #4d90fe, #357ae8);
	border: 1px solid #2f5bb7;
	color: #fff;
}

.button.create:hover,
.button.red:hover,
button.create:hover,
button.red:hover,
input[type="button"].create:hover,
input[type="button"].red:hover,
input[type="submit"].create:hover,
input[type="submit"].red:hover {
	background: -webkit-linear-gradient(top, #dd4b39, #c53727);
	background: -moz-linear-gradient(top, #dd4b39, #c53727);
	background: -ms-linear-gradient(top, #dd4b39, #c53727);
	background: -o-linear-gradient(top, #dd4b39, #c53727);
	background: linear-gradient(top, #dd4b39, #c53727);
	border: 1px solid #b0281a;
	border-bottom: 1px solid #af301f;
	-moz-box-shadow: 0 1px 1px rgba(0, 0, 0, .2);
	-webkit-box-shadow: 0 1px 1px rgba(0, 0, 0, .2);
	box-shadow: 0 1px 1px rgba(0, 0, 0, .2);
	color: #fff;
}

.button.green:hover,
.button.share:hover,
button.green:hover,
button.share:hover,
input[type="button"].green:hover,
input[type="button"].share:hover,
input[type="button"].green:hover,
input[type="submit"].share:hover {
	background: -moz-linear-gradient(top, #3d9400, #368200);
	background: -ms-linear-gradient(top, #3d9400, #368200);
	background: -o-linear-gradient(top, #3d9400, #368200);
	background: -webkit-linear-gradient(top, #3d9400, #368200);
	background: linear-gradient(top, #3d9400, #368200);
	border: 1px solid #2d6200;
	color: #fff;
	text-shadow: 0 1px rgba(0, 0, 0, .3);
}

.button:focus,
button:focus,
input[type="button"]:focus,
input[type="image"]:focus,
input[type="reset"]:focus,
input[type="submit"]:focus {
	-moz-box-shadow: inset 0 0 0 1px #fff;
	-webkit-box-shadow: inset 0 0 0 1px #fff;
	box-shadow: inset 0 0 0 1px #fff;
}

.button:active,
button:active,
input[type="button"]:active,
input[type="submit"]:active {
	background: -moz-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: -ms-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: -o-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: -webkit-linear-gradient(top, #f8f8f8, #f1f1f1);
	background: linear-gradient(top, #f8f8f8, #f1f1f1);
	border: 1px solid #ccc;
	-moz-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
	-webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
	box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
	color: #111;
}

.button.action:active,
.button.blue:active,
button.action:active,
button.blue:active,
input[type="button"].action:active,
input[type="button"].blue:active,
input[type="submit"].action:active,
input[type="submit"].blue:active {
	background: #357ae8;
	border: 1px solid #2f5bb7;
	-moz-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	-webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	color: #fff;
}

.button.create:active,
.button.red:active,
button.create:active,
button.red:active,
input[type="button"].create:active,
input[type="button"].red:active,
input[type="submit"].create:active,
input[type="submit"].red:active {
	background: -moz-linear-gradient(top, #dd4b39, #b0281a);
	background: -ms-linear-gradient(top, #dd4b39, #b0281a);
	background: -o-linear-gradient(top, #dd4b39, #b0281a);
	background: -webkit-linear-gradient(top, #dd4b39, #b0281a);
	background: linear-gradient(top, #dd4b39, #b0281a);
	border: 1px solid #992a1b;
	-moz-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	-webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	color: #fff;
}

.button.green:active,
.button.share:active,
button.green:active,
button.share:active,
input[type="button"].green:active,
input[type="button"].share:active,
input[type="submit"].green:active,
input[type="submit"].share:active {
	background: #368200;
	border: 1px solid #2d6200;
	-moz-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	-webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	box-shadow: inset 0 1px 2px rgba(0, 0, 0, .3);
	color: #fff;
}

.button.disabled,
button:disabled,
input[type="button"]:disabled,
input[type="image"]:disabled,
input[type="reset"]:disabled,
input[type="submit"]:disabled {
	background: #fff;
	border: 1px solid #dcdcdc;
	-moz-box-shadow: none;
	-webkit-box-shadow: none;
	box-shadow: none;
	color: #333;
	opacity: .5;
}

.button.action.disabled,
.button.blue.disabled,
button.action:disabled,
button.blue:disabled,
input[type="button"].action:disabled,
input[type="button"].blue:disabled,
input[type="submit"].action:disabled,
input[type="submit"].blue:disabled {
	background: -moz-linear-gradient(top, #4d90fe, #4787ed);
	background: -ms-linear-gradient(top, #4d90fe, #4787ed);
	background: -o-linear-gradient(top, #4d90fe, #4787ed);
	background: -webkit-linear-gradient(top, #4d90fe, #4787ed);
	background: linear-gradient(top, #4d90fe, #4787ed);
	border: 1px solid #3079ed;
	color: #fff;
}

.button.create.disabled,
.button.red.disabled,
button.create:disabled,
button.red:disabled,
input[type="button"].create:disabled,
input[type="button"].red:disabled,
input[type="submit"].create:disabled,
input[type="submit"].red:disabled {
	background: -moz-linear-gradient(top, #dd4b39, #d14836);
	background: -ms-linear-gradient(top, #dd4b39, #d14836);
	background: -o-linear-gradient(top, #dd4b39, #d14836);
	background: -webkit-linear-gradient(top, #dd4b39, #d14836);
	background: linear-gradient(top, #dd4b39, #d14836);
	border: 1px solid transparent;
	color: #fff;
}

.button.green.disabled,
.button.share.disabled,
button.green:disabled,
button.share:disabled,
input[type="button"].green:disabled,
input[type="button"].share:disabled,
input[type="submit"].green:disabled,
input[type="submit"].share:disabled {
	background: -moz-linear-gradient(top, #3d9400, #398a00);
	background: -ms-linear-gradient(top, #3d9400, #398a00);
	background: -o-linear-gradient(top, #3d9400, #398a00);
	background: -webkit-linear-gradient(top, #3d9400, #398a00);
	background: linear-gradient(top, #3d9400, #398a00);
	border: 1px solid #29691d;
	color: #fff;
	text-shadow: 0 1px rgba(0, 0, 0, .1);
}

.button + .button,
button + button,
input + input {
	margin-left: 12px;
}

/* CHECKBOXES */

input[type="checkbox"] {
	-webkit-appearance: none;
	background: #fff;
	border: 1px solid dimgray;
	border-image: -ms-linear-gradient(top, #fff, #fff);
	-webkit-border-radius: 1px;
	border-radius: 10px;
	font-size: smaller;
	height: 15px;
	left: 1px;
	margin: 2px 8px 0 0;
	outline: 0;
	position: relative;
	top: -2px;
	width: 15px;
}

input[type="checkbox"]:hover {
	-webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .1);
	box-shadow: inset 0 1px 1px rgba(0, 0, 0, .1);
	outline:none;
}

input[type="checkbox"]:active {
	background-color: #ebebeb;
	border: 1px solid #c6c6c6;
	outline: none;
}

input[type="checkbox"]:checked:after {
	background: url(https://ssl.gstatic.com/ui/v1/menu/checkmark_2x.png) no-repeat 0 0 / 21px;
	content: '';
	display: block;
	height: 21px;
	left: -5px;
	position: relative;
	top: -6px;
	width: 21px;
}

input[type="checkbox"]:disabled {
	background-color: #fff;
	border: 1px solid #e1e1e1;
	-webkit-box-shadow: none;
	box-shadow: none;
	opacity: .5;
}

input[type="text"] {
		background: none;
		width: 200px;
		height: 24px;
		line-height: 25px;
		font-size: 16px;
		border:0;
		border-bottom: 1px solid darkgray;
}

input[type="text"]:focus {
		outline:none!important;
}

label {
		color:gray;
		font-size:8pt;
}

w::-webkit-scrollbar {
		height: 12px;
		width: 3px;
		background: #fff;
}

w::-webkit-scrollbar-thumb {
		background: lightgray;
		-webkit-border-radius: 1ex;
		-webkit-box-shadow: 0px 1px 2px rgba(0, 0, 0, 0.75);
}

w::-webkit-scrollbar-corner {
		background: #000;
}

</style>
