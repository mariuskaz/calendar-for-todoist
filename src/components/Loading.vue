<template>
    <div class="loading">
		<div class="loading-circle"></div>
		<div class="loading-text">Loading</div>
	</div>
</template>

<style>
.loading {
	width: 120px;
	height: 120px;
	left:0;right:0;
	top:0;bottom:0;
	margin:auto;
	position:absolute;
	padding-left:250px;
 }

 .loading-circle {
	position:absolute;
	border-radius: 50%;
	border-top: 1px solid black;
	width: 120px;
	height: 120px;
	-webkit-animation: spin 2s linear infinite; /* Safari */
	animation: spin 1s linear infinite;
  }

  .loading-text {
    position: absolute;
    padding-top: 52px;
    text-align: center;
    font-family: roboto, sans-serif, "Helvetica Neue", "Lucida Grande", Arial;
    color: #303030;
    width: 120px;
    font-size: 11pt;
	text-transform:uppercase;
}

/* Safari */
@-webkit-keyframes spin {
	0% { -webkit-transform: rotate(0deg); }
	100% { -webkit-transform: rotate(360deg); }
}
  
@keyframes spin {
	0% { transform: rotate(0deg); }
	100% { transform: rotate(360deg); }
}

</style>