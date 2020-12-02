#### 常用特性
1、表单操作：
     input 单行文本
     textarea 多行文字
     select 下拉多选
     radio 单选框
     checkbox 多选框
     回顾： v-model双向绑定，通过data数据进行设置
                occupation：默认选择项
    
    表单域修饰符：
    number：转化为数值
    trim：去掉开始和结尾的空格
    lazy：将input事件切换为change事件
    <input v-model.number="age" type="number">
   

2、自定义指令
   语法规则：获取元素焦点
   ```js
   Vue.directive('focus' {
       inserted: function{
           //获取元素的焦点，借用vue的API：directive定义，第一个参数名称，第二个参数对象
           el.focus();
       }
   })
   ```
   自定义指令用法：
   <input type="text" v-focus>    用的时候前加v，声明的时候不加

   带参数的自定义指令（改变元素背景色）
      通过el绑定元素，直接来操作DOM
      binding：一个对象，包含一下属性：
            name：指令名，不包括 v- 前缀
            value：指令的绑定值
   ```js
   Vue.directive ('focus' , {
      inserted:function(el,binding) {
      el.style.backgroundColor = binding.value.color;
       }
   })
   ```
   指令的用法：  局部指令只在本组件中使用
   <input type="text" v-color='{color:"orange"}'>

   局部指令：
   ```js
   directive: {
       focus: {
           inserted:function (el) {
               el.focus();
           }
       }
   }
```

3、计算属性  让模板的算法变得简单
                     添加一个新的属性computed
```js
   computed: {
       reverseMessage: function () {
           return this.msg.split('').reverse().join('')
       }
   }
```
   计算属性与方法的区别：
      计算属性是基于他们的依赖进行缓存的
      方法不存在缓存


4、过滤器
   自定义过滤器  全局
   Vue.filter ('过滤器名称' , function (value)){
       //过滤器业务逻辑
   }
   过滤器使用
   <div>{{msg | upper}</div>  插值表达式
   <div v-bind:id="id | formatId"></div>    属性绑定  还可以级联操作  |lower |...
   局部过滤器   本组件使用
   filters:{
       capitalize :function(){}
   }
   案例时间：使用过滤器格式化日期
   ```js
  <div id="app">
    <div>{{date | format('yyyy-MM-dd hh:mm:ss')}}</div>
  </div>
  <script type="text/javascript" src="js/vue.js"></script>
  <script type="text/javascript">
    /*
      过滤器案例：格式化日期
      
    */
    // Vue.filter('format', function(value, arg) {
    //   if(arg == 'yyyy-MM-dd') {
    //     var ret = '';
    //     ret += value.getFullYear() + '-' + (value.getMonth() + 1) + '-' + value.getDate();
    //     return ret;
    //   }
    //   return value;
    // })
    Vue.filter('format', function(value, arg) {
      function dateFormat(date, format) {
          if (typeof date === "string") {
              var mts = date.match(/(\/Date\((\d+)\)\/)/);
              if (mts && mts.length >= 3) {
                  date = parseInt(mts[2]);
              }
          }
          date = new Date(date);
          if (!date || date.toUTCString() == "Invalid Date") {
              return "";
          }
          var map = {
              "M": date.getMonth() + 1, //月份 
              "d": date.getDate(), //日 
              "h": date.getHours(), //小时 
              "m": date.getMinutes(), //分 
              "s": date.getSeconds(), //秒 
              "q": Math.floor((date.getMonth() + 3) / 3), //季度 
              "S": date.getMilliseconds() //毫秒 
          };

          format = format.replace(/([yMdhmsqS])+/g, function(all, t) {
              var v = map[t];
              if (v !== undefined) {
                  if (all.length > 1) {
                      v = '0' + v;
                      v = v.substr(v.length - 2);
                  }
                  return v;
              } else if (t === 'y') {
                  return (date.getFullYear() + '').substr(4 - all.length);
              }
              return all;
          });
          return format;
      }
      return dateFormat(value, arg);
    })
    var vm = new Vue({
      el: '#app',
      data: {
        date: new Date()
      }
    });
  </script>
   ```

****
5、侦听器    watch属性
   数据一旦发生变化就通知侦听器所绑定方法
   应用场景：数据变化时执行异步或开销大的操作
```js
   watch: {
       firstName :function(val) {
           //val表示变化之后的值
           this .fullName = val +this.lastName;
       },
       lastName: function(val) {
           this.fullName=this.firstName+ val;
       }
   }
 ```
    案例时间： 验证用户名是否可用
       需求：输入框中输入姓名，**失去焦点**时验证是否存在，如果已经存在，提示从新输入，如果不存在，提示可以使用。

       需求分析：
       通过v-model实现数据双向绑定
       需求提供提示信息
       西药侦听器输入信息变化
       需要修改触发的时间
```js
 <div id="app">
        <div>
            <span>
        用户名：
    </span>
            <span>
                <input type="text v-model='uname '">
            </span>
            <span>{{tip}}</span>
        </div>

    </div>
    <script type="text/javascript" src="vue.js"></script>
    <script type="text/javascript">
        // 侦听器监听用户名的变化
        // 调用后台接口进行验证
        // 根据验证的结果调整提示信息
        var vm = new Vue({
            el: '#app',
            data: {
                uname: '',
                tip: '',
            },
            methods: {
                checkName: function(uname) {
                    // 调用接口，但是可以使用定时任务的方式模拟接口
                    setTimeout(function() {
                        var that = this;
                        if (uname == 'admin') {
                            that.tip = '用户名已经存在，请更换一个'
                        } else {
                            that.tip = '用户名可以使用'
                        }
                    }, 2000)
                }
            },
            watch: {
                uname: function(val) {
                    //调用后台接口验证用户名的合法性
                    this.checkName(val);
                    //修改提示信息
                    this.tip = '正在验证...'
                }
            }
        });
    </script>
```

6、生命周期
   beforeCreate：在实例初始化之后，数据观测和事件配置之前被调用 此时data 和 methods 以及页面的DOM结构都没有初始化   什么都做不了 

   created：在实例创建完成后被立即调用此时data 和 methods已经可以使用  但是页面还没有渲染出来 

   beforeMount：在挂载开始之前被调用   此时页面上还看不到真实数据 只是一个模板页面而已 

   **mounted**：el被新创建的vm.$el替换，并挂载到实例上去之后调用该钩子。  数据已经真实渲染到页面上  在这个钩子函数里面我们可以使用一些第三方的插件 

   beforeUpdate： 数据更新时调用，发生在虚拟DOM打补丁之前。   页面上数据还是旧的 
   updated：由于数据更改导致的虚拟DOM重新渲染和打补丁，在这之后会调用该钩子。 页面上数据已经替换成最新的
    
   beforeDestroy：实例销毁之前调用                        

   destroyed：实例销毁后调用



    数组变异方法
    在 Vue 中，直接修改对象属性的值无法触发响应式。当你直接修改了对象属性的值，你会发现，只有数据改了，但是页面内容并没有改变
    变异数组方法即保持数组方法原有功能不变的前提下对其进行功能拓展

   `push()`   往数组最后面添加一个元素，成功返回当前数组的长度            
   `pop()`      删除数组的最后一个元素，成功返回删除元素的值                 
   `shift()`    删除数组的第一个元素，成功返回删除元素的值                   
   `unshift()` | 往数组最前面添加一个元素，成功返回当前数组的长度             
   `splice()`   有三个参数，第一个是想要删除的元素的下标（必选），第二个是想要删除的个数（必选），第三个是删除 后想要在原位置替换的值 |
   `sort()`     sort()  使数组按照字符编码默认从小到大排序,成功返回排序后的数组
   `reverse()`  reverse()  将数组倒序，成功返回倒序后的数组                  

    替换数组
   不会改变原始数组，但总是返回一个新数组

    filter ： filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。 

    concat ： concat() 方法用于连接两个或多个数组。该方法不会改变现有的数组 
    slice：   slice() 方法可从已有的数组中返回选定的元素。该方法并不会修改数组，而是返回一个子数组 

   动态数组响应式数据
   Vue.set(a,b,c)    让 触发视图重新更新一遍，数据动态起来
   a是要更改的数据 、   b是数据的第几项、   c是更改后的数据

    案例时间：图书管理
```js  
<body>
    <div id="app">
        <div class="grid">
            <div>
                <h1>图书管理</h1>
                <div class="book">
                    <div>
                        <label for="id">
              编号：
            </label>
                        <input type="text" id="id" v-model='id' :disabled="flag" v-focus>
                        <label for="name">
              名称：
            </label>
                        <input type="text" id="name" v-model='name'>
                        <button @click='handle' :disabled="submitFlag">提交</button>
                    </div>
                </div>
            </div>
            <div class="total">
                <span>图书总数：</span>
                <span>{{total}}</span>
            </div>
            <table>
                <thead>
                    <tr>
                        <th>编号</th>
                        <th>名称</th>
                        <th>时间</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody>
                    <tr :key='item.id' v-for='item in books'>
                        <td>{{item.id}}</td>
                        <td>{{item.name}}</td>
                        <td>{{item.date | format('yyyy-MM-dd hh:mm:ss')}}</td>
                        <td>
                            <a href="" @click.prevent='toEdit(item.id)'>修改</a>
                            <span>|</span>
                            <a href="" @click.prevent='deleteBook(item.id)'>删除</a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
    <script type="text/javascript" src="js/vue.js"></script>
    <script type="text/javascript">
        /*
                      图书管理-添加图书
                    */
        Vue.directive('focus', {
            inserted: function(el) {
                el.focus();
            }
        });
        Vue.filter('format', function(value, arg) {
            function dateFormat(date, format) {
                if (typeof date === "string") {
                    var mts = date.match(/(\/Date\((\d+)\)\/)/);
                    if (mts && mts.length >= 3) {
                        date = parseInt(mts[2]);
                    }
                }
                date = new Date(date);
                if (!date || date.toUTCString() == "Invalid Date") {
                    return "";
                }
                var map = {
                    "M": date.getMonth() + 1, //月份 
                    "d": date.getDate(), //日 
                    "h": date.getHours(), //小时 
                    "m": date.getMinutes(), //分 
                    "s": date.getSeconds(), //秒 
                    "q": Math.floor((date.getMonth() + 3) / 3), //季度 
                    "S": date.getMilliseconds() //毫秒 
                };
                format = format.replace(/([yMdhmsqS])+/g, function(all, t) {
                    var v = map[t];
                    if (v !== undefined) {
                        if (all.length > 1) {
                            v = '0' + v;
                            v = v.substr(v.length - 2);
                        }
                        return v;
                    } else if (t === 'y') {
                        return (date.getFullYear() + '').substr(4 - all.length);
                    }
                    return all;
                });
                return format;
            }
            return dateFormat(value, arg);
        })
        var vm = new Vue({
            el: '#app',
            data: {
                flag: false,
                submitFlag: false,
                id: '',
                name: '',
                books: []
            },
            methods: {
                handle: function() {
                    if (this.flag) {
                        // 编辑图书
                        // 就是根据当前的ID去更新数组中对应的数据
                        this.books.some((item) => {
                            if (item.id == this.id) {
                                item.name = this.name;
                                // 完成更新操作之后，需要终止循环
                                return true;
                            }
                        });
                        this.flag = false;
                    } else {
                        // 添加图书
                        var book = {};
                        book.id = this.id;
                        book.name = this.name;
                        book.date = 2525609975000;
                        this.books.push(book);
                        // 清空表单
                        this.id = '';
                        this.name = '';
                    }
                    // 清空表单
                    this.id = '';
                    this.name = '';
                },
                toEdit: function(id) {
                    // 禁止修改ID
                    this.flag = true;
                    console.log(id)
                        // 根据ID查询出要编辑的数据
                    var book = this.books.filter(function(item) {
                        return item.id == id;
                    });
                    console.log(book)
                        // 把获取到的信息填充到表单
                    this.id = book[0].id;
                    this.name = book[0].name;
                },
                deleteBook: function(id) {
                    // 删除图书
                    // 根据id从数组中查找元素的索引
                    // var index = this.books.findIndex(function(item){
                    //   return item.id == id;
                    // });
                    // 根据索引删除数组元素
                    // this.books.splice(index, 1);
                    // -------------------------
                    // 方法二：通过filter方法进行删除
                    this.books = this.books.filter(function(item) {
                        return item.id != id;
                    });
                }
            },
            computed: {
                total: function() {
                    // 计算图书的总数
                    return this.books.length;
                }
            },
            watch: {
                name: function(val) {
                    // 验证图书名称是否已经存在
                    var flag = this.books.some(function(item) {
                        return item.name == val;
                    });
                    if (flag) {
                        // 图书名称存在
                        this.submitFlag = true;
                    } else {
                        // 图书名称不存在
                        this.submitFlag = false;
                    }
                }
            },
            mounted: function() {
                // 该生命周期钩子函数被触发的时候，模板已经可以使用
                // 一般此时用于获取后台数据，然后把数据填充到模板
                var data = [{
                    id: 1,
                    name: '三国演义',
                    date: 2525609975000
                }, {
                    id: 2,
                    name: '水浒传',
                    date: 2525609975000
                }, {
                    id: 3,
                    name: '红楼梦',
                    date: 2525609975000
                }, {
                    id: 4,
                    name: '西游记',
                    date: 2525609975000
                }];
                this.books = data;
            }
        });
    </script>
</body>
```

