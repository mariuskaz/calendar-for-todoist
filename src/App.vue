<template>
	<div id="app">
		<heading :days='days' :status='status' :translate='translate' @today='today' @view='redraw' @sync="sync" />
		<sidebar :persons='persons' :status='status' :tasks="tasks" :translate='translate' @sync='sync' />
		<calendar :tasks='tasks' :days='days' :status='status' :translate='translate' @addTask="addTask" @dropTask="dropTask" />
		<connect v-if="status.connect" :status="status" :input='input' :translate='translate' @add='addUser' />
		<task v-if="status.taskId > 0" :status="status" :tasks='tasks' :details="details" :projects='projects' :notes='notes' :users='users' :translate='translate' @updateTask="updateTask"/>
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

			dictionary: {

				'en': {
					add: 'Add',
					connect: 'Connect',
					close: 'Close',
					new: 'New task',
					tasks: 'Tasks',
					today: 'Today',
					token: 'Todoist API token',
					user: 'User',
					refresh: 'Refresh',
					weekDays: [ 'Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday' ]
				},

				'lt': {
					add: 'Pridėti',
					connect: 'Įtraukti',
					close: 'Baigti',
					new: 'Nauja užduotis',
					tasks: 'Užduotys',
					today: 'Šiandien',
					token: 'Todoist raktas',
					user: 'Darbuotojas',
					refresh: 'Atnaujinti',
					weekDays: [ 'sekmadienis', 'pirmadienis' , 'antradienis', 'trečiadienis', 'ketvirtadienis', 'penktadienis', 'šeštadienis' ]
				},

			},

			days: [],
			tasks: [],
			details: {},
			projects: {},
			notes: [],
			users: {},

		}
	},

	computed: {
		translate() {
			let lang = navigator.language.substring(0,2).toLowerCase(),
			dict = this.dictionary[lang] || this.dictionary['en']
			return dict
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

		redraw(d = 1) {

			this.days = this.days.map(day => {
					let date = new Date(day)
					date.setDate(date.getDate() + d)
					return date.toLocaleDateString()
			})

			if (d < -1 || d > 1) 
				this.status.view += 1

		},

		sync(wipe = false) {
		
			if (wipe) this.tasks = []
			if (this.status.person == 0) return
			
			let view = this,
			id = this.status.person,
			todoist = new Todoist()
			todoist.token = this.persons[id].token || 'none'
			todoist.sync.oncomplete = function(data) {
				view.update(data, wipe)
			}

			console.log('sync...') 
			this.status.sync = true
			if (wipe) this.status.busy = true
			todoist.sync()

		},

		update(data, permanent = false) {

			let user = 'user' + data.user.id,
			items = localStorage[user] ? JSON.parse(localStorage[user]) : {}

			for (let item in items) 
				items[item].completed = true

			data.items = { ...items, ...data.items }
			localStorage.setItem(user, JSON.stringify(data.items))
			
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
					let stored = JSON.parse(localStorage[item.id])
					duration = stored.duration || 30
				}

				if (item.responsibleUid == this.status.userId 
					|| item.projectId == this.status.inboxId) {

					let desc = item.content
					if (desc.indexOf('[') == 0 && desc.indexOf('](' != -1)) 
						desc = desc.split('](')[0].substring(1)

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
					let delta = 
						new Date(a.due.date) - new Date(b.due.date)
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

		addUser() {

			this.persons.push({ name: this.input.user, token: this.input.token, color: this.input.color  })
			this.status.person = this.persons.length - 1
			localStorage.setItem('persons', JSON.stringify(this.persons))
			this.sync()
			this.input.user = ''
			this.input.token = ''
			
		},

		getDay(day = 0) {

			let date = new Date
			return new Date(date.setDate(date.getDate() + day)).toLocaleDateString()

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
