### 变量

在golang渲染template的时候，可以接受一个interface{}类型的变量，我们在模板文件中可以读取变量内的值并渲染到模板里。

有两个常用的传入参数的类型。一个是struct，在模板内可以读取该struct域的内容来进行渲染。还有一个是map[string]interface{}，在模板内可以使用key来进行渲染。

我一般使用第二种，效率可能会差一点儿，但是用着方便。

模板内内嵌的语法支持，全部需要加{{}}来标记。

在模板文件内， . 代表了当前变量，即在非循环体内，.就代表了传入的那个变量。假设我们定义了一个结构体：

```
type Article struct {
    ArticleId int
    ArticleContent string
}
```

那么在模板内可以通过

```
<p>{{.ArticleContent}}<span>{{.ArticleId}}</span></p>
```

来获取并把变量的内容渲染到模板内。假设上述的结构体的内容为ArticleId:1 ArticleContent:”hello”， 则对应渲染后的模板内容为：

```
<p>hello<span>1</span></p>
```

当然，我们有时候需要定义变量，比如我们需要定义一个article变量，同时将其初始化为”hello”，那么我们可以这样写：

```
{{$article := "hello"}}
```

假设我们想要把传入值的内容赋值给article，则可以这样写：

```
{{$article := .ArticleContent}}
```

这样我们只要使用{{$article}}则可以获取到这个变量的内容。

### 函数

golang的模板其实功能很有限，很多复杂的逻辑无法直接使用模板语法来表达，所以只能使用模板函数来绕过。

首先，template包创建新的模板的时候，支持.Funcs方法来将自定义的函数集合导入到该模板中，后续通过该模板渲染的文件均支持直接调用这些函数。

该函数集合的定义为：

```
type FuncMap map[string]interface{}
```

key为方法的名字，value则为函数。这里函数的参数个数没有限制，但是对于返回值有所限制。有两种选择，一种是只有一个返回值，还有一种是有两个返回值，但是第二个返回值必须是error类型的。这两种函数的区别是第二个函数在模板中被调用的时候，假设模板函数的第二个参数的返回不为空，则该渲染步骤将会被打断并报错。

在模板文件内，调用方法也非常的简单：

```
{{funcname .arg1 .arg2}}
```

假设我们定义了一个函数

```
func add(left int, right int) int
```

则在模板文件内，通过调用

```
{{add 1 2}}
```

### 判断

golang的模板也支持if的条件判断，当前支持最简单的bool类型和字符串类型的判断

```
{{if .condition}}
{{end}}
```

当.condition为bool类型的时候，则为true表示执行，当.condition为string类型的时候，则非空表示执行。

当然也支持else ， else if嵌套

```
{{if .condition1}}
{{else if .contition2}}
{{end}}
```

假设我们需要逻辑判断，比如与或、大小不等于等判断的时候，我们需要一些内置的模板函数来做这些工作，目前常用的一些内置模板函数有：

- not 非

  {{if not .condition}}
  {{end}}

- and 与

  {{if and .condition1 .condition2}}
  {{end}}

- or 或

  {{if or .condition1 .condition2}}
  {{end}}

- eq 等于

  {{if eq .var1 .var2}}
  {{end}}

- ne 不等于

  {{if ne .var1 .var2}}
  {{end}}

- lt 小于 (less than)

  {{if lt .var1 .var2}}
  {{end}}

- le 小于等于

  {{if le .var1 .var2}}
  {{end}}

- gt 大于

  {{if gt .var1 .var2}}
  {{end}}

- ge 大于等于

  {{if ge .var1 .var2}}
  {{end}}

### 循环

golang的template支持range循环来遍历map、slice内的内容，语法为：

```
{{range $i, $v := .slice}}
{{end}}
```

在这个range循环内，我们可以通过

v来访问遍历的值，还有一种遍历方式为：

```
{{range .slice}}
{{end}}
```

这种方式无法访问到index或者key的值，需要通过.来访问对应的value

```
{{range .slice}}
{{.field}}
{{end}}
```

当然这里使用了.来访问遍历的值，那么我们想要在其中访问外部的变量怎么办？(比如渲染模板传入的变量)，在这里，我们需要使用$.来访问外部的变量

```
{{range .slice}}
{{$.ArticleContent}}
{{end}}
```

### 模板的嵌套

在编写模板的时候，我们常常将公用的模板进行整合，比如每一个页面都有导航栏和页脚，我们常常将其编写为一个单独的模块，让所有的页面进行导入，这样就不用重复的编写了。

任何网页都有一个主模板，然后我们可以在主模板内嵌入子模板来实现模块共享。

当模板想要引入子模板的时候，我们使用以下语句：

```
{{template "navbar"}}
```

这样子就会尝试载入名称为navbar的子模板，同时我们也得定义一个子模板来实现”navbar”这个子模板。

子模板的定义为：

```
{{define "navbar"}}
{{end}}
```

在定义之间的内容将会覆盖{{template “navbar”}}

当然子模板是分离了，那么子模板能否获得父模板的变量呢？这是当然的，我们只需要使用

```
{{template "navbar" .}}
```

就可以将当前的变量传给子模板了，这个也是相当方便的。

原文链接：https://blog.csdn.net/tianlongtc/article/details/80073098



相关文章：

https://studygolang.com/articles/21435?fr=sidebar

https://studygolang.com/articles/3071