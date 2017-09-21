<template>
	<div id="app">
		<todo></todo>
	</div>
</template>

<script >
    // 对应着不同的版本
//	import Todo from './todo.vue';
//	import Todo from './todo-2.vue';
	import Todo from './todo-3.vue';
    
    export default {
        name: 'app',
        components: {
            Todo
        }

    }
</script>

<style>

html * {
    margin: 0;
    padding: 0;
}
body {

	background-color: #328e8e;
	color: #fff;
}


</style>
































