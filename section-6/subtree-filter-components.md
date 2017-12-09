# 6.2 子树过滤器
子树过滤器由XML元素及其XML属性组成。 在子树过滤器中有五种类型的组件：

 - 名字空间选择(`Namespace Selection`)

 - 属性匹配表达式(`Attribute Match Expressions`)

 - 包含节点(`Containment Nodes`)

 - 选择节点(`Selection Nodes`)

 - 内容匹配节点(`Content Match Nodes`)

## 6.2.1 命名空间选择

如果`<filter>`元素中与特定节点关联的`XML`名称空间与底层数据模型中的名称空间相同，则认为名称空间匹配（用于过滤目的）。

> 请注意，名称空间选择本身不能使用。 如果要在过滤器输出中包含任何元素，至少必须在过滤器中指定一个元素。

为子树筛选定义了`XML`名称空间通配符机制。 如果`<filter>`元素中的一个元素没有被一个名称空间限定（例如，`xmlns =""`），那么当处理该子树过滤器节点时，服务器必须评估它所支持的所有`XML`名称空间。 这种通配符机制不适用于`XML`属性。

> 请注意，在将过滤器元素与底层数据模型中的元素进行比较时，限定名称空间的前缀值不相关。

例子：

```xml
<filter type="subtree">
    <top xmlns="http://example.com/schema/1.2/config"/>
</filter>
```

在这个例子中，`<top>`元素是一个选择节点，只有“`http://example.com/schema/1.2/config`”命名空间中的这个节点和任何子节点（来自底层数据模型）将被包括在过滤器输出。

## 6.2.2 属性匹配表达式

出现在子树过滤器中的属性是“属性匹配表达式”的一部分。任何数量的（非限定或限定的）XML属性都可以出现在任何类型的过滤器节点中。除了通常适用于该节点的选择标准之外，所选数据必须具有节点中指定的每个属性的匹配值。如果一个元素没有被定义为包含一个指定的属性，那么它在筛选器输出中就不会被选中。

例：

```xml
<filter type="subtree">
  <t:top xmlns:t="http://example.com/schema/1.2/config">
    <t:interfaces>
      <t:interface t:ifName="eth0"/>
    </t:interfaces>
  </t:top>
</filter>
```

在这个例子中，`<top>`和`<interfaces>`元素是包含节点，`<interface>`元素是一个选择节点，而“`ifName`”是一个属性匹配表达式。只有 “`http://example.com/schema/1.2/config`” 命名空间中具有值“`eth0`”且出现在“`top`”节点内的“`interfaces`”节点内的“`ifName`”属性的“`interface`”节点将包含在过滤器输出中。

## 6.2.3 包含节点

在子树过滤器中包含子元素的节点称为“包含节点”。 每个子元素可以是任何类型的节点，包括另一个包含节点。 对于在子树过滤器中指定的每个包含节点，所有与指定的命名空间，元素层次结构和所有属性匹配表达式完全匹配的数据模型实例都包含在过滤器输出中。

例：

```xml
<filter type="subtree">
    <top xmlns="http://example.com/schema/1.2/config">
        <users/>
    </top>
</filter>
```

在这个例子中，`<top>`元素是一个包含节点。

## 6.2.4 选择节点

过滤器中的空叶节点称为“选择节点”，它表示底层数据模型上的“显式选择”过滤器。在一组兄弟节点中存在任何选择节点将导致过滤器选择指定的子树并且抑制基础数据模型中的整个兄弟节点集合的自动选择。为了过滤目的，可以用空标签（例如，`<foo />`）或者用明确的开始和结束标签（例如，`<foo> </foo>`）来声明空叶节点。任何空白字符在这种形式下被忽略。

例：

```xml
<filter type="subtree">
  <top xmlns="http://example.com/schema/1.2/config">
    <users/>
  </top>
</filter>
```

在这个例子中，`<top>`元素是一个包含节点，`<users>`元素是一个选择节点。只有“`http://example.com/schema/1.2/config`”命名空间中的“`users`”节点出现在配置数据存储根目录的`<top>`元素内才会包含在过滤器输出中。

## 6.2.5 内容匹配节点

包含简单内容的叶节点称为“内容匹配节点”。它用于选择部分或全部兄弟节点作为过滤器输出，它表示叶节点元素内容上的精确匹配过滤器。以下约束适用于内容匹配节点：

- 内容匹配节点不能包含嵌套元素。

- 多个内容匹配节点（即兄弟节点）在逻辑上以“`AND`”表达式组合。

- 不支持过滤混合内容。

- 不支持过滤列表内容。

- 不支持仅对空白内容进行过滤。

- 内容匹配节点必须包含非空白字符。空元素（例如`<foo> </foo>`）将被解释为选择节点（例如`<foo />`）。

- 前导和尾随的空格字符被忽略，但是文本字符块中的任何空白字符都不会被忽略或修改。

如果所有指定的兄弟内容都匹配子树中的节点，则表达式中的节点为“`true`”，那么将按照以下方式选择筛选器输出节点：

- 兄弟集中的每个内容匹配节点都包含在筛选器输出中。

- 如果在兄弟集合中存在任何包含节点，那么如果任何嵌套的过滤条件也被满足，则它们被进一步处理并被包括在内。

- 如果兄弟集合中存在任何选择节点，则它们都包含在筛选器输出中。

- 如果选择节点的任何兄弟节点是概念数据结构（例如，列表关键节点(`list key leaf`)）的实例标识符组件，则它们也可以被包括在过滤器输出中。

- 否则（即在筛选器兄弟集合中没有选择或包含节点），在基础数据模型中的此级别定义的所有节点（及其子树，如果有的话）将在筛选器输出中返回。

如果任何兄弟内容匹配节点测试为“`false`”，则不对该兄弟集执行进一步的过滤处理，并且包括内容匹配节点的过滤器不选择兄弟子树。

例：

```xml
<filter type="subtree">
  <top xmlns="http://example.com/schema/1.2/config">
    <users>
      <user>
        <name>fred</name>
      </user>
    </users>
  </top>
</filter>
```
在本例中，`<users>`和`<user>`节点都是包含节点，`<name>`是内容匹配节点。由于没有指定`<name>`的兄弟节点（因此没有包含或选择节点），所以`<name>`的所有兄弟节点都将在过滤器输出中返回。只有匹配元素层次结构并且`<name>`元素等于“`fred`”的“http://example.com/schema/1.2/config”名称空间中的“用户”节点才会包含在过滤器输出中。