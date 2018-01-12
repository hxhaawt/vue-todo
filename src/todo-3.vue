<template>
	<div id="todo" class="todo-app">
		
        <header class="header">
            <h1>todos</h1>
            <input  class="new-todo"
                placeholder="新的todo"
                v-model="newTodo"
                @keyup.enter="addTodo"
            />
        </header>
        
        <div class="main" v-show="pageTodoList.length">
            <ul class="todo-list">
                <li v-for="(todo, index) in pageTodoList"
                    :key="index"
                    @dblclick="editTodo(todo)"
                    class='todo'
                    
                >
                    <div :class="['view', {'editing': todo == editedTodo}]">
                        <input type="checkbox"
                               class="toggle"
                               v-model="todo.completed"
                        />
                        <span :class="['title', {completed: todo.completed}]"
                              @dblclick="editTodo(todo)"
                              :title="todo.title"
                              v-html="todo.title"
                        >
                            <!--{{ todo.title}}-->
                        </span>
                        <button class="destroy" @click="removeTodo(todo)">x</button>
                    </div>
                    
                    <div :class="['waitEdit', {'edit': todo == editedTodo}]"
                         contenteditable="true"
                         v-focus="todo == editedTodo"
                         ref="edit"
                         v-html="todo.title"
                         @blur="doneEdit(todo)"
                         @keyup.esc="cancelEdit(todo)"
                    >
                        <!--{{ todo.title }}-->
                    </div>
                    
                </li>
            </ul>
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
                    <!--<button @click="openDialog('isShowClearAllTodo')">-->
                    <button @click="openAllTodoDialog">
                        清除所有任务
                    </button>
                    <!--<button @click="openDialog('isShowClearCompleteTodo')">-->
                    <button @click="openCompleteTodoDialog">
                        清除已完成任务
                    </button>
                </div>
            </div>
        </footer>
        
        <div class="dialog-position">
            <my-dialog :is-show="isShowClearCompleteTodo"
    
                       @on-close="closeDialog('isShowClearCompleteTodo')">
                <div class="clear-dialog">
                    <div class="title">清除已经完成任务</div>
                    <button class="cancel"
                            @click="closeDialog('isShowClearCompleteTodo')">
                        取消
                    </button>
                    <button class="sure" @click="sureClearCompleteTodo">
                        确认
                    </button>
                </div>
            </my-dialog>
            <my-dialog :is-show="isShowClearAllTodo"
                       @on-close="closeDialog('isShowClearAllTodo')">
                <div class="clear-dialog">
                    <div class="title">清除所有任务</div>
                    <button class="cancel"
                            @click="closeDialog('isShowClearAllTodo')">
                        取消
                    </button>
                    <button class="sure" @click="sureClearAllTodo">
                        确认
                    </button>
                </div>
            </my-dialog>
        </div>
       
	</div>
</template>

<script >
    
    import Dialog from './components/dialog.vue';
    
    // 外部过滤器
    var filters = {
        all: todos => todos,
        active: todos => todos.filter(todo => !todo.completed),
        completed: todos => todos.filter(todo => todo.completed)
    };
    const STORAGE_KEY = 'todo-v1.2-key';
    
    var todoStorage = {
        fetch () {
            // parse用于从一个字符串中解析出json对象
            // todos是对象
            var storeItem = localStorage.getItem(STORAGE_KEY);

            var todos = JSON.parse(storeItem || '[{"title":"请输入您的todo任务", "completed":false}]');

            // 由于空对象转换成boolean类型时也是true，所以如果想在todo
            // 页面中始终都有一条todo显示，就要像下面一样
            // 不然像上面一样，当用户删除了所有todo时，下面的提示就不
            // 会显示了，如果只要显示一次提示todo，可以用上面的
            // var todos = JSON.parse(storeItem).length>0 ? JSON.parse(storeItem) : JSON.parse('[{"title":"请输入您的todo任务", "completed":false}]');

            todos.forEach((todo, index) => { todo.id = index });
            todoStorage.id = todos.length;

            return todos;
        },
        save (todos) {
            // 以字符串的形式保存
            // stringify()用于从一个对象解析出字符串
            console.log("save-->");
            console.log(todos);
            localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
        }
    };
    
    export default {
        name: 'todo',
        components: {
            MyDialog: Dialog
        },
        data() {
            return {
                isShowClearCompleteTodo: false,
                isShowClearAllTodo: false,  // 显示确认清除所有todo弹窗
                isChecked: false,
                beforeEditCache: '',    // 缓存编辑前的todo内容
                editedTodo: null,   // 保存正在编辑的todo，是个对象
                newTodo: '',        // 保存输入的todo内容
                visibility: 'all',  // 全局过滤器选择
                todoList: ""        // 获取本地存储的todo列表
            }
        },
        // 生命周期函数
        mounted () {
            // 获取本地存储的todo列表
            this.todoList = todoStorage.fetch();
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
                        completed: false,
                        id: todoStorage.id++
                    });
                    this.newTodo = "";
                }
                console.log("1-->");
                console.log(this.todoList);
            },
            removeTodo (todo) {
                this.todoList.splice(this.todoList.indexOf(todo), 1)
            },
            editTodo(todo) {
                this.beforeEditCache = todo.title;
                this.editedTodo = todo;
    
                this.$refs.edit[todo.id].innerHTML = '';
                this.$refs.edit[todo.id].innerHTML = todo.title;
            },
            // 完成编辑
            doneEdit (todo) {
                if (!this.editedTodo) return;
                this.editedTodo = null;

                todo.title = this.$refs.edit[todo.id].innerHTML;
                
                todo.title = todo.title.trim();
                if (!todo.title) this.removeTodo(todo);
            },
            // 取消编辑
            cancelEdit (todo) {
                this.editedTodo = null;
                todo.title = this.beforeEditCache;
            },
            
            // 设置当前的过滤器
            setVisibility(param) {
                this.visibility = param;
            },
            
            // 显示 确认 清除已完成任务弹窗
            openCompleteTodoDialog() {
//                console.log('22' +  filters.completed(this.todoList).length );
                if ( filters.completed(this.todoList).length > 0) {
                    this.isShowClearCompleteTodo = true;
                }
            },
            
            // 显示 确认 清除全部任务弹窗
            openAllTodoDialog() {
//                console.log( '11' + this.todoList.length );
                if ( this.todoList.length > 0) {
                    this.isShowClearAllTodo = true;
                }
            },
            
            // 清除已经完成的任务
            sureClearCompleteTodo() {
                this.todoList = filters.active(this.todoList);
                this.isShowClearCompleteTodo = false;
            },
            // 清除清除所有todo任务
            sureClearAllTodo() {
                this.todoList = [];
                this.isShowClearAllTodo = false;
            },
            
            // 显示弹窗
            openDialog (attr) {
                this[attr] = true;
            },
            
            // 关闭弹窗
            closeDialog (attr) {
                this[attr] = false;
            },
    
        },
        
        filters: {
            // 内部过滤器
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

<style lang="scss" scoped>

/*  用sass写样式 */

/* 让vue 有数据时才显示*/
[v-cloak] {
    display:none;
}

$todo-app-width: 800px;
$font-color: #328e8e;

.todo-app {
    max-width: $todo-app-width;
    margin-top: 100px;
    margin-left: auto;
    margin-right: auto;
    position: relative;
}

.header {
    /*background-color: red;*/
    h1 {
        width: 100px;
        margin: 0 auto;
        font-size: 100px;
    }
    
    input.new-todo {
        height: 40px;
        width: 100%;
    
        box-sizing: border-box;
        margin-top: 20px;
        padding-left: 10px;
        border: 1px solid #328e8e;
        border-radius: 5px;
    }
}

.main {
    
    margin-top: 16px;
    background-color: $font-color;
    
    ul{
        list-style-type: none;
        margin: 0;
        padding: 0;
    }
    
    ul li {
        position: relative;
        
        margin-bottom: 8px;
        
        width: $todo-app-width;
        
        font-size: 20px;
        
        border: 1px solid #328e8e;
        border-radius: 5px;
        background-color: #fff;
        
        color: $font-color;
        
        cursor: pointer;
        
        .view {
            vertical-align: middle;
        }
        .editing {
            display: none;
        }
    
        .toggle {
            display: block;
            position: absolute;
            left: 10px;
            top: 50%;
            margin-top: -15px;
            
            width: 20px;
            height: 30px;
            
            cursor: pointer;
    
            transition: all .3s;
        }
        /* todo内容展示 样式*/
        .title {
            display: inline-block;
            
            line-height: 30px;
            margin: 8px 0 8px 40px;
            
            width: 700px;
            
            cursor: pointer;
            
            word-wrap:break-word;
        }
        /* 已经完成的 todo 样式 */
        .completed {
            color: #d9d9d9;
            text-decoration: line-through;
        
            transition: all 0.5s ease-out;
        }

        /* 删除按钮样式*/
        .destroy {
            display: block;
            position: absolute;
            right: 20px;
            top: 10px;
            border: none;
            font-size: 20px;
            background-color: #fff;
            color: #fff;
            cursor: pointer;
            
            transition: all 0.5s ease-out;
        }
    
        &:hover {
            .destroy {
                color: #cc9a9a;
            }
        }
    
        .waitEdit {
            display: none;
        }
        .edit {
            
            display: block;
            padding: 8px;
            line-height: 30px;
            min-height: 40px;
            cursor: auto;
            /*width: 780px;*/
            
            word-wrap: break-word;
            /*white-space: pre-wrap;*/
            
            outline:0;
            
            color: $font-color;
            
            &:focus {
                box-shadow: 0 0 8px blue;
            }
        }
    
    }
}


/*列表底部样式*/
.todo-footer {
    position: relative;
    color: #328e8e;
    
    .footer {
        
        position: fixed;
        top: 242px;
        right: 10px;
        z-index: 10;
        
        margin: 0;
        box-sizing: border-box;
        width: 180px;
        
        border: 1px solid #328e8e;
        border-radius: 5px;
        
        padding-top: 10px;
    
        background-color: #fff;
    }
    
    span {
        padding-left: 20px;
    }
    ul, li {
        cursor: pointer;
    }
    ul {
        list-style-type: none;
        background-color: #fff;
        margin-left: 20px;
        margin-top: 20px;
        margin-bottom: 10px;
    
        li {
            padding: 4px;
            width: 100px;
            margin-top: 10px;
            &:hover {
                border: 1px solid blue;
                border-radius: 3px;
            }
    
            &.selected {
                border: 1px solid blue;
                border-radius: 3px;
            }
        }
    }
    
    .clear-todo {
        margin-top: 30px;
        margin-left: 20px;
        margin-bottom: 20px;
    
        button {
            cursor: pointer;
            margin-bottom: 10px;
            padding: 4px;
            width: 130px;
            border: none;
            
            text-align: left;
            background-color: #fff;
            color: #328e8e;
            
            &:hover {
                border: 1px solid blue;
                border-radius: 3px;
            }
        }
    }
    
}
/* 响应式样式 */
@media only screen and (max-width: 1300px) {
    
    .todo-footer {
        /*position: relative;*/
        /*color: #328e8e;*/
        
        .footer {
    
            position: fixed;
            top: auto;
            right: auto;
            letf: auto;
            bottom: 0;
            box-sizing: border-box;

            /*border-radius: 0px;*/
            
            margin: 0;
            padding: 0;
            
            width: $todo-app-width;
            height: 50px;
            
            /*background-color: #fff;*/
        }
        
        ul, li {
            /*cursor: pointer;*/
            margin: 0;
            display: inline-block;
        }
        ul {
            /*list-style-type: none;*/
            margin-left: 16px;
            /*background-color: #fff;*/
            
            li {
                padding: 4px;
                width: 80px;
                /*margin-top: 10px;*/
                /*&:hover {*/
                    /*border: 1px solid blue;*/
                    /*border-radius: 3px;*/
                /*}*/
            }
        }
        
        .clear-todo {
            margin: 0 0 0 10px;
            
            display: inline-block;
        }
        
    }
    
}


/*  所有弹窗的位置都在浏览器最底部*/
.dialog-position {
    /*position: fixed;*/
    /*bottom: -10px;*/
}
/* 弹窗内部数据样式 */
.clear-dialog {
    width: 200px;
    margin: 10px auto;
    overflow: hidden;
    
    .title {
        
        margin-bottom: 10px;
        /*text-align: left;*/
        font-size: 20px;
        color: $font-color;
    }
    .cancel,
    .sure {
        width: 60px;
        height: 40px;
        color: $font-color;
    }
    .cancel {
        margin-right: 20px;
    }
}




</style>
































