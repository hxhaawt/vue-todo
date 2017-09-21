<template>
	<div id="todo">
		<div class="container">
            <h1>todos</h1>
            <div class="todo-input">
                <input placeholder="新的todo"
                       v-model="newTodo"
                       @keyup.enter="addTodo"
                />
                <button @click="addTodo">submit</button>
            </div>
            <div class="todo-view">
                <div v-for="(todo, index) in pageTodoList"
                     :key="index"
                >
                    <div :class="['todo-list',
                        {'todo-select': todo.completed, editing: todo == editedTodo}]"
                         @dblclick="editTodo(todo)"
                    >
                        <input type="checkbox"
                               v-model="todo.completed" />
                        <label @dblclick="editTodo(todo)"> {{ todo.title }}</label>
                        <button class="destroy" @click="removeTodo(todo)">x</button>
                    </div>
                    
                    <input :class="['waitEdit', {'edit': todo == editedTodo}]" type="text"
                           v-model="todo.title"
                           @blur="doneEdit(todo)"
                           v-focus="todo == editedTodo"
                           @keyup.enter="doneEdit(todo)"
                           @keyup.esc="cancelEdit(todo)"
                    />
                </div>
                <footer class="todo-footer">
                    <div class="footer">
                        <span>
                        {{ displayChart }} {{ remaing }} 个
                        </span>
                        <ul>
                            <li :class="{ selected: visibility == 'all' }"
                                @click="setVisibility('all')">全部任务</li>
                            <li :class="{ selected: visibility == 'active' }"
                                @click="setVisibility('active')">等待完成</li>
                            <li :class="{ selected: visibility == 'completed' }"
                                @click="setVisibility('completed')">已经完成</li>
                        </ul>
                        <div class="clear-todo">
                            <button @click="clearAllTodo">清除所有任务</button>
                            <button @click="clearCompleteTodo">清除已完成任务</button>
                        </div>
                    </div>
                </footer>
            </div>
		</div>
	</div>
</template>

<script >
    // 外部过滤器
    var filters = {
        all: todos => todos,
        active: todos => todos.filter(todo => !todo.completed),
        completed: todos => todos.filter(todo => todo.completed)
    };
    const STORAGE_KEY = 'todo-v1.0-key';
    
    var todoStorage = {
        fetch () {
            var todos = JSON.parse(localStorage.getItem(STORAGE_KEY)
                || '[{"title":"请输入您的todo任务", "completed":false}]');
            
//            todos.forEach((todo, index) => { todo.id = index });
//            todoStorage.uid = todos.length;
            return todos;
        },
        save (todos) {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
        }
    };
    
    export default {
        name: 'todo',
        components: {

        },
        data() {
            return {
                isChecked: false,
                beforeEditCache: '',    // 缓存编辑前的todo内容
                editedTodo: null,   // 保存正在编辑的todo，是个对象
                newTodo: '',        // 保存输入的todo内容
                visibility: 'all',  // 全局过滤器选择
                todoList: todoStorage.fetch()   // 获取本地存储的todo列表
            }
        },
        watch: {
            todoList: {
                handler: todoList => todoStorage.save(todoList),
                deep: true
            }
        },
        computed: {
            /* 不同选择对应todo个数*/
            remaing () {
                // return filters.active(this.todoList).length;
                return filters[this.visibility](this.todoList).length;
            },
            // 显示在左边todo的文字
            displayChart() {
              return  this.visibility == 'all' ? '全部任务'
                        :  this.visibility == 'active' ? '等待完成任务' : '已经完成任务';
            },
            // 显示在页面上的todo
            pageTodoList () {
                return filters[this.visibility](this.todoList)
            }
            
        },
        methods: {
    
            addTodo() {
                if (this.newTodo) {
                    this.todoList.push({
                        title: this.newTodo.trim(),
                        completed: false
                    });
                    this.newTodo = "";
                }
            },
            removeTodo (todo) {
                this.todoList.splice(this.todoList.indexOf(todo), 1)
            },
            editTodo(todo) {
                this.beforeEditCache = todo.title;
                this.editedTodo = todo;
            },
            doneEdit (todo) {
                if (!this.editedTodo) return;
                this.editedTodo = null;
                todo.title = todo.title.trim();
                if (!todo.title) this.removeTodo(todo);
            },
            cancelEdit (todo) {
                this.editedTodo = null;
                todo.title = this.beforeEditCache;
            },
            
            setVisibility(param) {
                this.visibility = param;
            },
            
            // 清除所有todo任务
            clearAllTodo() {
                this.todoList = [];
            },
            // 清除已经完成的任务
            clearCompleteTodo() {
              this.todoList = filters.active(this.todoList);
            }
    
        },
        
        filters: {
        
        },

        directives: {
            focus: function (el, {value}){
                if ( value ) {
                    el.focus()
                }
            }
        }

    }
</script>

<style scoped>

/*[v-cloak] {*/
    /*display:none !important;*/
/*}*/

.container {
    position: relative;
	max-width: 900px;
	margin: 50px auto;
}

.container h1 {
    width: 100px;
    margin: 20px auto;
    font-size: 100px;
}
.todo-input {
    height: 40px;
}
.todo-input input {
    /*margin: 0;*/
	width: 800px;
	height: 100%;
    box-sizing: border-box;
    padding-left: 10px;
    border: 1px solid #328e8e;
    border-radius: 5px;
}

.todo-input button {
    margin: 0;
    /*float: right;*/
	width: 95px;
	height: 100%;
    text-align: center;
    background-color: #fff;
    border: 1px solid #94dfdf;
    border-radius: 5px;
}
.todo-input button:hover {
    background-color: #94dfdf;
    color: #fff;
}

.todo-view {
    position: relative;
    margin-top: 20px;
}


.todo-list{
    
    position: relative;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    margin-bottom: 10px;
    padding-left: 8px;
    
    min-height: 36px;
    line-height: 36px;
    border-radius: 5px;
    background-color: #fff;
    color: #328e8e;
    cursor: pointer;
    transition: all .3s;
}
.todo-list label {
    cursor: pointer;
}
.todo-select {
    color: #d9d9d9;
    text-decoration: line-through;
}

.editing {
    display: none;
}


/* 删除todo btn 样式*/
.destroy {
    display: none;
    position: absolute;
    right: 20px;
    bottom: 5px;
    border: none;
    font-size: 20px;
    background-color: #fff;
    color: #cc9a9a;
    
    transition: all 0.3s ease-out;
}

.todo-list:hover .destroy{
    color: #cc9a9a;
    display: block;
}



.todo-list input[type=checkbox] {
    margin-right: 6px;
}

.waitEdit {
    display: none;
}
.edit {
    display: block;
    box-sizing: border-box;
    margin-bottom: 10px;
    padding-left: 8px;
    
    width: 100%;
    min-height: 36px;
    line-height: 36px;
    border: 1px solid #fff;
    border-radius: 5px;
    background-color: #fff;
    color: #328e8e;
}


/*列表底部样式*/
.todo-footer {
    position: relative;
}
.footer {
    
    position: fixed;
    top: 242px;
    right: 10px;
    
    border: 1px solid #328e8e;
    border-radius: 5px;
    
    width: 180px;
    background-color: #fff;
    color: #328e8e;
    padding-top: 10px;
}
.footer span {
    padding-left: 20px;
}
.footer ul,
.footer li {
    cursor: pointer;
    /*line-height: 34px;*/
    /*display: inline-block;*/
}
.footer ul {
    list-style-type: none;
    margin-left: 20px;
    margin-top: 20px;
    margin-bottom: 10px;
}
.footer ul li {
    padding: 4px;
    width: 100px;
    margin-top: 10px;
    /*margin-right: 20px;*/
}
.footer ul li:hover {
    border: 1px solid blue;
    border-radius: 3px;
}
.selected {
    border: 1px solid blue;
    border-radius: 3px;
}


.clear-todo {
    margin-top: 30px;
    margin-left: 20px;
    margin-bottom: 20px;
}

.clear-todo button {
    cursor: pointer;
    margin-bottom: 10px;
    padding: 4px;
    width: 130px;
    border: none;
    
    background-color: #fff;
    color: #328e8e;
    
    text-align: left;
}

.clear-todo button:hover {
    border: 1px solid blue;
    border-radius: 3px;
}

@media only screen and (max-width: 1300px) {
    .footer {
        
        position: absolute;
        bottom: 0 !important;
        margin: 0;
        padding: 0;
        width: 900px;
        height: 50px;
    }
    .footer ul,
    .footer li {
        margin: 0;
        /*line-height: 34px;*/
        display: inline-block;
    }
    
    .clear-todo {
        margin: 0;
        display: inline-block;
    }
}

</style>
































