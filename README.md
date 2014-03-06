# tablegrid
	对于动态获取数据的场景，使用标准table完成的grid。
	优点是你可以任意定义table的样式，目前版本是基于bootstrap的实现，未来会将他们分开。
	总的思路就是让配置信息尽可能的在html里完成，js只负责事件和行为。
	优点是平衡了html与js的代码量，避免了html只有一个div其它的全是js的情况，也减少了总代码量。
	让html专注显示，让js专注行为。
	由于tablegrid使用单个table进行渲染，因此无法实现滚动功能的grid。

# 1.目标
- 高度
	- 根据渲染数据的行数，自动计算高度
	- 根据指定行数，自动计算高度
	- 按照指定高度（px），滚动显示数据
- 表头
	- 默认使用单行表头
	- 支持复杂表头
	- 支持第一列全选功能
- 表尾
	- 默认单行
	- 支持多行（翻页+新建）
- 样式
	- tablegrid不关注样式
- 行为
	- 支持翻页
	- 支持缓存数据
	- 支持本地数据和远程数据
- 方法
	- 行操作
	- 列操作

# 2.标签
## table
- tablegrid:			指定了tablegrid属性的table标签自动初始化。
- data_type：		数据格式，目前只支持json
- data_provider： 	varname|{url:'index.php', method:'get'}
- data_reader：		
	
	- undefined |'' : 从根开始
	- { root: 'root' } : 从root开始
	- { root: 'root', total: 'total' } ： 指定root和记录总数

- chkallbox：	是否启用首列的 checkbox。
- multiselect：	是否允许多选
- autoLoad：	是否自动加载数据
- height: 5|100px

## thead
	thead用于定义表头，优先读取usebind=true属性的tr用于绑定，没有usebind属性时读取第一行作为绑定行。

## th
- label: 指定列的标题
- bind:
	- string: 绑定使用的字段
	- null string: 使用模板
- style: 列使用的样式
- class: 列使用的class
- header_style: 表头使用的样式
- header_class: 表头使用的class

**简单方式**

	<th label="title1" bind="field1"></th>
**模板方式**

	<th label="title1">
	    <input type="text" name="aa"/>
	</th>
	注：不指定bind属性即为模板方式

**复杂表头的列绑定**

	对于复杂表头tablegrid要求必须使用thead标签，使用.usebind作为绑定行

	<table>
		<thead>
			<tr>
				<th colspan=2>总计1</th>
				<th colspan=2>总计2</th>
			</tr>
			<tr usebind=true>
				<th label="列1" bind="field_name1"/>
				<th label="列2" bind="field_name2"/>
				<th label="列3" bind="field_name3"/>
				<th label="列4" bind="field_name4"/>
			</tr>
		</thead>
	</table>

效果如下：
<table>
	<thead>
		<tr>
			<th colspan=2>总计1</th>
			<th colspan=2>总计2</th>
		</tr>
		<tr>
			<th>列1</th>
			<th>列2</th>
			<th>列3</th>
			<th>列4</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>field1_value</td>
			<td>field2_value</td>
			<td>field3_value</td>
			<td>field4_value</td>
		</tr>
	</tbody>
</table>

**group(分组)**

	感谢jquery，她为我画group提供了一个新思路，方便很多，以前自己画group太bt了。
	如果不是行模板渲染的方式，可以通过为列指定group属性进行分组。
	分组操作比较当前cell内容与上一行同列cell位置的值，如果相同则进行分组。
	使用很简单，设置 group = true 或 group = "true" 即可。
	分组可以使用其他列，设置 group = "another_field"。

	<th label="t1" bind='field1" group="true|another_field"/>


### 翻页属性
待完善

- page：		是否翻页，page=0不翻页，其它表示翻页时每页显示的最大记录数
- toolbar：		是否启用toobar，目前看本参数意义不大，因为按钮由html代码设定
- records：		是否显示records信息，false不显示，其它都为显示
	
## tfoot标签
	tfoot标签提供使用一个colspan值与table列数相同的td作为功能条，功能条包含下面三个子功能条。
	由于功能条涉及到展现的事，将展现从tablegrid中分离出来我还没想好，所以目前版本还是使用bootstrap的样式。
### toolbar
	样式是使用方式参考bootstrap的文档，需要明确的就是工具条里面的按钮是单独绑定事件的，与tablegrid没有任何关系。
	这件事我也挺纠结，理论上应该在一起，但是在一起真的不方便。
	toobar必须自己在html中编写。
### pagebar
	你只需要在tabel标签中指定page=10和jsonReader即可，tablegrid会来处理翻页按钮的。

### records信息
	同pagebar，指定records=true即可。
## 样式
	tablegrid使用了如下样式

- table.tablegrid :		tablegrid会据此自动加载样式为tablegrid的table。
- table>thead>tr.usebind :	复杂表头时，据此选择用来绑定的tr。
- table>tfoot>tr>td:first.table-grid-gridbar : tablegrid的功能条类
- table>tfoot>tr>td:first.table-grid-gridbar div.table-grid-toolbar : tablegrid的工具条
- table>tfoot>tr>td:first.table-grid-gridbar div.table-grid-pagination : tablegrid的翻页条
- table>tfoot>tr>td:first.table-grid-gridbar div.table-grid-records-info :	tablegrid的信息条


# 3.js
## 初始化
## options
	事件在html中定义非常不方便，因此提供js方式。
	options具有table标签的全部定义功能，并且options会覆盖table标签里的定义。
## 事件
- post：		动态加载时提交的数据。
- oncomplete：	tablegrid 渲染完成时被触发。
- onselrow：	当一个数据行被选中时触发。
- ontrcomplete:	当一个tr渲染后被触发。


## 方法
	tablegrid提供如下访问接口。

- reloadData 	： 刷新数据
- page 			： 翻页
- getSelected 	： 取得选中行的索引，从1开始
- getCell 		： 取得指定行和列名称的列数据，没有进行bind的也能取到，只要在server返回的数据内即可。
- getSelectedCell： 取得选中的cell数据，如选中多行则返回数组
- getSelectedRow ： 取得选中的行，如多行则返回数组
- getCol 		： 取得指定列的数组

## 问题
- 如何添加行内按钮？

	tablegrid将功能按钮放在toobar上了，对行内按钮支持的不好，但你可以使用 列模板的渲染方式来实现

---
- 注：tablegrid 需要依赖 jquery 库，如果使用template进行渲染，还需要underscore.js库。
- 注：th、td都被tablegrid识别，但建议使用th。