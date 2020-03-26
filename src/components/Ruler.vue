<template>
    <div class="line">
        <div class="buble"></div>
    </div>
</template>

<script>
export default {
    name: 'ruler'
}
</script>

<style>
.line {
    border-bottom:2px solid #ea4335;
    display:inline-block;
    height:2px;
    width:100%;
    position:absolute;
    margin-top:20px;
    z-index:9999;
}
.buble {
    background:#ea4335;
    border-radius:6px;
    height:12px;
    width:12px;
    margin-left:-6px;
    margin-top:-3px;
}
</style>