title: 国地税联合办税表单引擎
speaker:  基础办税
url: http://foresee.com.cn
transition: cards
theme: moon
highlightStyle:

[slide]
#国地税联合办税表单引擎
<small>Beta</small>

[slide data-transition="bounceIn"]
*  表单引擎介绍
	* plugin-rule-analyse.js --计算规则表达式解析 {:&.moveIn}
	* plugin-rule-engine.js --执行规则表达式
	* plugin-tip.js --提示信息
	* plugin-util.js --工具
	* plugin-json.js --json解析相关
	* plugin-logger.js --提示信息
	* plugin-money-format.js --工具
	* plugin-validate.js --校验规则
	* plugin-view-bind.js --数据绑定
	* plugin-view-drow.js --动态行处理
	* plugin-function.js --自定义函数（规则表达式已支持自定义函数校验和计算）
[slide data-transition="bounceIn"]
* framework.js
*       /**异步请求data跟rule后初始化视图及绑定事件*/
		```
	    $.when(
            $.getJSON(settings.rulesurl),
            $.getJSON(settings.dataurl),
            $.getJSON(settings.validateurl)
        ).then(function(rules,data,validate){
            settings.data = data[0];
       	    settings.rules = rules[0]; //规则
            settings.validate = validate[0]; //校验
        	if (data[0] == null) {
        		$("body").unmask();
        		Message.errorInfo({
    				title : "提示", message : "申报表未初始化，请使用【手工初始化】功能初始化申报表。"
    			});
        		return;
        	}
		}.done() {
		}
		```
* 只有当所有when成功聚集之后，返回promise对象，成功时执行then(),done()方法始终会被执行
[slide data-transition="vertical3d"]
* CommonJS规范、AMD规范
    * 使用require.js实现对模块化功能的定义和加载
    * define：定义模块
    ```
    define(['jquery', 'plugin-util', 'plugin-rule-analyse', 'plugin-view-bind', 'plugin-json'], 
	    function ($, util, analyse, bind, pluginjson) {
		/**css样式处理*/
		function classProcess(target,operator){
			if(operator=="add"){
			   $(target).addClass("yellow");
		   } else if(operator=="remove"){
		       $(target).removeClass("yellow");
		   }
		}
	    /**对外提供接口**/
	    return{
	    	isEmpty:isEmpty,
	    	hasTip:hasTip,
	    	deleteTip:deleteTip,
	    	buildTip: buildTip,
	    	removeTip:removeTip,
	    	classProcess:classProcess
	    };
	});
    ```
[slide data-transition="vertical3d"]
    * require：加载模块
```
require(['jquery', 'plugin-util', 'plugin-rule-engine', 'plugin-view-bind', 'plugin-json','plugin-validate','plugin-tip','plugin-view-drow', 'mask', 'smartmenu'], 
function ($,util, engine, bind, pluginjson,validate,tip,drow) {
/*数据绑定*/
bind.mapping(view, eval(realDataName));
/*初始化提示*/
initializeTip(view,dataName);
/*绑定input输入事件*/
view.find("input").each(function () {
    $(this).on("input", function () {
    	autoChnageMoney(this);
        var path = dataName + "." + $(this).attr(settings.datatarget);//属性ID
        var value = util.toNumber($(this).val()); //值强转为number
        if(util.notEmpty($(this).attr(settings.datatarget))) {//获取同名input索引 如果没有jpath则不处理
            var index = view.find(util.format("input[jpath='{0}']",
                $(this).attr(settings.datatarget).replace(".", "\\."))).index(this);
            pluginjson.setValue(settings.data, path, value, index);//更新json对象中的值
            if (dataValidate(this,path,index)) { /*校验通过执行规则*/                        
                engine.excute(view, settings.data, settings.rules, settings.validate,path, dataName);//执行规则
                validate.process(settings.data,settings.validate,this,path,index);
                initializeTip(view,dataName);//联动校验其他元素
            }
        }
   });
});
 ```
[slide data-transition="moveIn"]
## 表单开发过程
* 准备工作 {:&.moveIn}
* 得到一份表单原型(xxx.html) 
* 一份金三xsd格式下的json字符串
* 根据json层级路径，绑定节点到相应单元格（格式为jpath="x.x.x.x.x"）
* 配置nssb_dzbd(类似金三的nf_xtgl_bdmbdy)
* 配置规则表达式,并生成规则文件
* 
[slide  data-transition="moveIn"]
[magic data-transition="vertical3d"]
##表单效果如下
----
[增值税纳税申报表(适用于增值税小规模纳税人)](https://github.com/andylieonian/myppt/blob/master/image/bd.png?raw=true)
<img src="https://github.com/andylieonian/myppt/blob/master/image/bd.png?raw=true" class="img-responsive">
====
## 动态行处理
* 表单中的动态行，这里以小规模中附表《增值税减免税申报明细表》为例说明
```
//动态行所在table上加样式 
<table class="zzssyyxgmnsrySbSbbdxxVO.zzsjmssbmxb jmmxb"> zzssyyxgmnsrySbSbbdxxVO.zzsjmssbmxb
var tr = view.find("table."+key.replace(/\./g,"\\.")).find("tr");//根据样式来找到动态行所在位置
<tr group="-1">//动态行所在table -><tr>加自定义标签group,用来标识哪一个动态行，
若只有一个动态行，group="-1"；
/*配置完html,还需要在plugin-view-drow.js中配置动态行，关键字、行内元素默认值、序号、表头行数、
表尾行数等信息*/
	var jsonData = "";
	/**申报表动态行效果处理**/
	var jmmxb = new Array("", "xh", 0, 0, 0, 0, 0, "", "xh", 0, 0, 0, 0, 0, 0);
	var fjs = new Array("", "", 0, 0, 0, 0, 0, 0, 0, 0, 0, 0);
	/*动态行，关键字、行内元素默认值、序号、表头行数、表尾行数*/
	var dthJsonData = 
	[{key:"hkyscdd", defaultVals:hkyscdd, xh:-1, thead:8, ttail:0},
	 {key:"zzssyyxgmnsrySbSbbdxxVO.zzsjmssbmxb", defaultVals:jmmxb, xh:1, thead:4, ttail:1},
	  {key:"hXZGSB01285Request.fjsSbbdxxVO.fjssbb", defaultVals:fjs, xh:1, thead:4, ttail:1}];
```
[/magic]
[slide data-transition="moveIn"]
## select处理
```
/*配置html单元格,mode="mix"表示下拉框以dm|mc 形式显示；jpath为dm在json中路径，xpath为码表路径，
zpath为mc在json中路径*/
<select mode="mix" jflag="1" jpath="zzsjmssbmxbjsxmGrid.zzsjmssbmxbjsxmGridlbVO.dm" 
 xpath="../xml/nf_dm_sb_zzs_jsxzdm.xml" zpath="zzsjmssbmxbjsxmGrid.zzsjmssbmxbjsxmGridlbVO.hmc">
</select>
```
## 根据代码带出对应名称的处理
```
/*配置html单元格,jpath为dm在json中路径，xpath为码表路径，
zpath为mc在json中路径*/
<input jflag="1" type="text" datatype="string" jpath="sbxxGrid.sbxxGridlbVO.zsxmDm" 
xpath="../xml/dm_gy_zsxm.xml" zpath="sbxxGrid.sbxxGridlbVO.zsxmMc">
```
> 表单使用smartMenu.css来制作右键增加/删除行（屏蔽自带的右键菜单，自己生成）
----
[slide  data-transition="moveIn"]
[magic data-transition="vertical3d"]
##增加/删除效果如下
----
[增值税减免税申报明细表](https://github.com/andylieonian/myppt/blob/master/image/dth.png?raw=true)
<img src="https://github.com/andylieonian/myppt/blob/master/image/dth.png?raw=true" class="img-responsive">
##**QA**
<small>谢谢大家，欢迎大家踊跃提问</small>