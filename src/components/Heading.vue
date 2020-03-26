<template>
  <div class="header">
    <img style="height:30px;vertical-align:middle;margin-right:8px" src="todoist.png">
    <a style="margin-right:40px;colorr:#E44332" href="https://todoist.com" target="_blank">todoist</a>
    <button @click="$emit('today')" class="aaction" style="margin-right:20px;vertical-align:middle">Šiandien</button>
    <i @click="$emit('back',7)" class="fa fa-angle-double-left big-icon"></i>
    <i @click="$emit('back')" class="fa fa-angle-left big-icon"></i>
    <span style="margin:0 10px">{{ year }} {{ month }}</span>
    <i  @click="$emit('forward')" class="fa fa-angle-right big-icon"></i>
    <i @click="$emit('forward',7)" class="fa fa-angle-double-right big-icon"></i>
    <i @click="$emit('sync')" class="fa fa-refresh big-icon" title="Atnaujinti" style="float:right;margin-right:35px;"></i>
    
  </div>  
</template>

<script>
export default {
    props: ['days'],
    name: 'heading',

    computed: {
      year() {
        return new Date(this.days[2]).getFullYear()
      },
      month() {
        const day = new Date(this.days[2])
        return day.toLocaleString('default', { month: 'long' })
      }
    },

    updated() {
        document.title = "Todoist kalendorius - " + this.year + " " + this.month
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
</style>