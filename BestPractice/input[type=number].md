## input[type=number]
1. 仅能输入数字 小数点 负号
2. 小数点仅能输入一次，负号仅能输入两次，位置不限定。超出次数无法输入，也不会触发input事件

显示值ViewValue
显示值为输入框当前所显示的值，下面显示值以ViewValue代替
ViewValue与element.value并不绝对相等。

element.value
1. 字符串。
2. ViewValue以小数点结尾时，小数点会被忽略。
3. ViewValue整体不合法时为空字符串。

element.valueAsNumber
1. 数字。等于Number(ViewValue)
2. ViewValue以小数点结尾时，为整数部分的值。
3. ViewValue整体不合法时为NaN。

element.validationMessage
原生的校验信息。ViewValue有问题时有相关提示信息。

input事件：
1. 在每一次输入时触发，包括小数点和负号。（即只在ViewValue改变时触发）
2. e.data 为当前输入的单个字符，包括小数点和负号

change事件
1. 输入框聚焦 || 框内按下enter键时，将当前ele.value作为初始值
2. (输入框失焦 || 再次按下enter键) && (当前ele.value与初始值不同) 时触发
3. 根据1，给数字添加一个小数点后失焦，不会触发change事件


## el-input[type=number]

显示值ViewValue
显示值为输入框当前所显示的值，下面显示值以ViewValue代替。
ViewValue与bindValue并不绝对相等。

bindValue
1. 与ele.value相等。
2. 根据1，bindValue为字符串。
2. 根据1，ViewValue以小数点结尾时，bindValue不会更新（因为在原生input给一个数添加小数点结尾时，ele.value并不会改变）。
3. 根据1，ViewValue整体不合法时为空字符串。

input事件：
1. 在 (每一次输入 && bindValue改变) 时触发（即只在bindValue改变时触发）
2. e = bindValue

change事件
1. 输入框聚焦 || 框内按下enter键时，将当前bindValue作为初始值
2. (输入框失焦 || 再次按下enter键) && (当前bindValue与初始值不同) 时触发
3. 根据1，给数字添加一个小数点后失焦，不会触发change事件


## UniApp input[type=number]
1. 仅能输入数字 小数点 负号
2. 禁止小数点在首位输入，不会触发任何事件和效果
2. 小数点仅能输入一次，负号仅能输入两次，位置不限定。超出次数无法输入，也不会触发input事件
4. 负号输入不会触发input事件
5. 非法值在失焦后会直接清空

显示值ViewValue
显示值为输入框当前所显示的值，下面显示值以ViewValue代替
ViewValue与e.detail.value并不绝对相等。

e.detail.value
1. 字符串。
2. ViewValue以小数点结尾时，小数点会被忽略。
3. ViewValue整体不合法时为空字符串。

bindValue
bindValue与e.detail.value相等。

input事件：
1. 在每一次输入为非负号，且引发ViewValue改变时触发。

change事件
没有change事件


## UniApp中，输入框限制输入位数时，必须把修改的值放在nextTick里的原因
以限制输入3位小数为例：
1. 初始值为1.333
2. 再输入一位：1.3334
3. 被限制，将绑定值重新赋值为1.333

这个过程中，绑定值的初始值和最终值是相等的，不会触发Vue更新视图，所以若第3步不在nextTick中，那么输入框将显示1.3334，而实际绑定值已经被更新为1.333。
为什么在Pc端没有这个问题？
输入在显示更新之前
1. 输入框输入1.333
2. 输入看输入1.3334
3. 触发vmodel的原生input事件，input将绑定值更新为1.3334
4. 触发绑定的限制输入input事件，将绑定值更新为1.333