<template>
  <div class="header">
    <img style="height:30px;vertical-align:middle;margin-right:8px" src="todoist.png">
    <a style="margin-right:40px;colorr:#E44332" href="https://todoist.com" target="_blank">todoist</a>
    <button @click="$emit('today')" class="aaction" style="margin-right:20px;vertical-align:middle">Šiandien</button>
    <i @click="$emit('view',-7)" class="fa fa-angle-double-left big-icon"></i>
    <i @click="$emit('view',-1)" class="fa fa-angle-left big-icon"></i>
    <span style="margin:0 10px">{{ year }} {{ month }}</span>
    <i  @click="$emit('view',1)" class="fa fa-angle-right big-icon"></i>
    <i @click="$emit('view',7)" class="fa fa-angle-double-right big-icon"></i>
    <i @click="$emit('sync')" class="fa fa-repeat big-icon right" :class="{ 'fa-spinner': status.sync, 'fa-pulse': status.sync }" title="Refresh"></i>
		<div style="float:right;margin:2px 6px;font-size:x-small;color:gray;line-height:14px;text-align:center">Last sync:<br><b>{{ status.lastSync }}</b></div>
  </div>  
</template>

<script>
export default {
    props: ['days','status'],
    name: 'heading',

    computed: {
    
      year() {
        return new Date(this.days[2]).getFullYear()
      },

      month() {
        const today = new Date,
        day =  today.getDay(),
        date = new Date(this.days[day - 1])
        return date.toLocaleString('default', { month: 'long' })
      }
      
    },

    updated() {
        document.title = "Todoist kalendorius"
    },
}
</script>

<style>
.header {
    position:fixed;
    background:#fafafa;
    color: #3c4043;
    padding: 10px;
    font-family: 'Google Sans', Roboto, Arial, sans-serif;
    font-size: 22px;
    font-weight: 400;
    letter-spacing: 0;
    line-height: 28px;
    white-space: nowrap;
    width:100%;
    top:0;
    user-select:none;
  }

  .big-icon {
    border:1px solid white;
    color:gray;
    font-size:22px;
    line-height:22px;
    cursor: pointer;
    margin:0;
    padding:4px;
  }

  .big-icon:hover {
    border:1px solid #F3F3F3;
    background:whitesmoke;
  }

  .right {
		float:right; 
		margin-right:30px;
    	border-radius: 16px;
    	width: 22px;
    	text-align: center;
	}

	.spinner {
		/* background: LavenderBlush; */
		animation: spin 0.6s linear infinite;
	}

	@keyframes spin {
		0% { transform: rotate(0deg); }
		100% { transform: rotate(360deg); }
	}

</style>