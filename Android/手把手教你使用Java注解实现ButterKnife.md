# 手把手教你使用Java注解实现ButterKnife（待完成...）

#### Java内置的3种标准注解

<table class="table table-striped">
	<tr>
		<td>@Override</td>
		<td><h6>表示当前的方法定义将覆盖超类中的方法。如果你不小心拼写错误，或者方法签名对不上被覆盖的方法，编译器就会发出错误提示。</h6></td>
	</tr>
	<tr>
		<td>@Deprecated</td>
		<td><h6>如果程序员使用了注解为它的元素，那么编译器就会发出警告信息。</h6></td>
	</tr>
	<tr>
		<td>@Deprecated</td>
		<td><h6>关闭不当的编译器警告信息。</h6></td>
	</tr>
</table>

#### Java内置的4种元注解

<table class="table table-striped">
	<tr>
		<th>元注解</th>
		<th>解释</th>
	</tr>
	<tr>
		<td>@Target</td>
		<td><h6>表示该注解可以用于什么地方，可能的ElementType参数包括：</h6>
		<strong>CONSTRUCTOR：</strong>构造器的声明 <br/>
		<strong>FIELD：</strong>域声明（包括enum实例）<br/>
		<strong>LOCAL_VARIABLE：</strong>局部变量声明 <br/>
		<strong>METHOD：</strong>方法声明 <br/>
		<strong>PACKAGE：</strong>包声明 <br/>
		<strong>PARAMETER：</strong>参数声明 <br/>
		<strong>TYPE：</strong>类、接口（包括注解类型）或enum声明
		</td>
	</tr>
	<tr>
		<td>@Retention</td>
		<td><h6>表示需要在声明级别保存该注解信息。可选的RetentionPolicy参数包括：</h6>
		<strong>SOURCE：</strong>注解将被编译器丢弃<br/>
		<strong>CLASS：</strong>注解在class文件中可用，但会被VM丢弃<br/>
		<strong>RUNTIME：</strong>VM将在运行期也保留注解，因此可以通过反射机制读取注解的信息<br/>
		</td>
	</tr>
	<tr>
		<td>@Documented</td>
		<td><h6>将此注解包含在JavaDoc中</h6></td>
	</tr>
	<tr>
		<td>@Inherited</td>
		<td><h6>允许子类继承父类的注解</h6></td>
	</tr>
</table>

