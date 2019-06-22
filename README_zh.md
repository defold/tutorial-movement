# 移动教程

这是给初学者的教程。它引导你去创建一个玩家控制的飞碟（太空飞船），并且当你使用按键操纵它时，它的移动方式显得很自然真实。当你完成本教程，你会了解下列问题的答案：

* 什么是向量（vectors）？
* 如何使用向量表示位置、速度与加速度？
* 如何使用它创建一个游戏 Demo？以此为基础进行实验和延伸开发。

本文假设您对物理概念（如速度和加速度）有基本的了解。您还需要对 Lua 语言有一些基本的了解。

这个项目是为你预设好的，所以没有任何前置步骤。通过 [运行这个游戏](defold://build) 来预览它的内容：

- 它包含了图形：一个会动的飞船和一张背景图。
- 对方向键与鼠标点击的输入预设。
- 一个名为 "spaceship" 游戏对象，以及附带脚本。
- 脚本具有适当的代码以响应输入行为。目前，它只会将输入的信息输出到 console（控制台）。

当这个游戏运行，尝试按箭头键或点击屏幕，然后查看 IDE 的控制台来找找脚本输出的“输入信息”。提醒：根据你在游戏内运行不同的按键，你可能看到不同的文本：

<img src="doc/input.png" srcset="doc/input@2x.png 2x">

## 让飞船动起来

在深入了解细节之前，让我们先做个简单实践：让飞船原地浮动。

打开 ["spaceship.script"](defold://open?path=/main/spaceship.script) 脚本，并找到 `on_input()` 函数的代码：

```lua
function on_input(self, action_id, action)
    if action_id == hash("up") then
        print("UP!")
    elseif...
```

你看到作为示例的 `print("UP!")` 代码了吗？将其替换为如下代码：

```lua
function on_input(self, action_id, action)
    if action_id == hash("up") then
        local p = go.get_position()
        p.y = p.y + 1
        go.set_position(p)
    elseif...
```

再次 [运行游戏](defold://build) 并按下 <kbd>方向上键 ↑</kbd>，观察它向上的运动行为。这代码非常简单，不过别掉以轻心，让我们来看看这段代码做了什么：

```lua
if action_id == hash("up") then
```

我们通过项目内的“按键绑定”（此文件：["/input/game.input_binding"](defold://open?path=/input/game.input_binding)），来预设每个方向键的动作名："up"、"down"、"left"以及"right"。这个游戏以每秒 60 帧的速度播放，并逐帧加载你按键触发的“up”动作，这个 [散列](https://en.wikipedia.org/wiki/Hash_function) 过的"up"动作名会被发送到 `on_input()` 函数里。当你按住按钮时，`if` 至首个 `elseif` 之间的语句，就会一秒执行 60 次。

```lua
local p = go.get_position()
```

`go.get_position()` 函数获取游戏对象的位置（对象）。当不传参数的调用它时，返回的就是游戏对象的 *当前* 位置。这段代码属于游戏对象“太空飞船”，自然返回的是“太空飞船”的位置。

位置对象被赋值给了局部变量 `p`，由此可以通过操作 `p` 来更改坐标。位置对象使用了 `vector3` 类，它是含三个坐标属性的 *向量*。

```lua
p.y = p.y + 1
```

`p` 通过对象 `vector3` 描述了一个位于 3D 空间的点，这个点由 X、Y、Z 坐标构成。按下 <kbd>上键</kbd> 应沿 Y 轴移动，`y` 属性作为位置，应当加 1。

```lua
go.set_position(p)
```

最终，当前的游戏对象接收了新的位置改动。

在继续之前，让我们来试试改变 `p.y` 的值，你可以试试从 1 到 5，并再次 [运行游戏](defold://build)。感受一下现在飞船的移动速度吧。

最后，在 `go.set_position(p)` 代码下增加一行以输出 `p` 的值：

```lua
function on_input(self, action_id, action)
    if action_id == hash("up") then
        local p = go.get_position()
        p.y = p.y + 5
        go.set_position(p)
        print(p)
    elseif...
```

再次 [运行游戏](defold://build)，现在你可以逐帧查看表明位置的向量值，它由引擎打印出来。注意，向量的第二个值将随飞船移动而变化，因为它是 `y` 轴值：

```text
...
DEBUG:SCRIPT: vmath.vector3(640, 460, 0)
DEBUG:SCRIPT: vmath.vector3(640, 465, 0)
DEBUG:SCRIPT: vmath.vector3(640, 470, 0)
DEBUG:SCRIPT: vmath.vector3(640, 475, 0)
...
```

## 向量

向量是具备 _方向_ 与 _数量_ （或长度）的数学实体。它描述了向量空间内的特定点。在实际中，一个向量内含了一组数字，它们负责标识这个点的坐标。在二维空间（平面）内，描述一个向量必须要两个数：分别对应 X、Y 轴：

<img src="doc/2d-vector.png" srcset="doc/2d-vector@2x.png 2x">

在三维空间，同理，你需要三个数字来描述：X、Y 与 Z 轴。

<img src="doc/3d-vector.png" srcset="doc/3d-vector@2x.png 2x">

关于向量 *v* 的数量或长度，往往使用“毕达哥拉斯定理”（勾股定理）来计算：

<img src="doc/magnitude.png" srcset="doc/magnitude@2x.png 2x">

当向量的数量为 1 时，此时它被称为 *单位向量*（或标准化的向量）。

尽管 Defold 是个为 2D 游戏定制的工具集，它也的确是 3D 引擎。所有的游戏对象和组件的位置都处于三维空间，其位置通过 `vector3` 对象来表示。当你查看你的 2D 游戏时，X、Y 轴的值表示游戏对象在“宽度”（左右）与“高度”（上下）的位置，Z 轴的值决定了“深度”位置。位置 Z 值允许你控制重叠在一起的对象可见性：Z 值为 1 的精灵将位于 Z 值为 0 的精灵的前面。默认情况下，Defold 使用的坐标系允许 Z 值介于 -1 到 1 之间：

<img src="doc/z-order.png" srcset="doc/z-order@2x.png 2x">

Defold 的 Lua 库 [`vmath`](https://defold.com/ref/vmath) 中，包含了创建与操作 [`vector3`](https://defold.com/ref/vmath/#vmath.vector3) 对象的方法:

```lua
-- 创建一个新的 vector3 对象，并预设 X 轴位置为 100，Y 轴位置为 350。
local position = vmath.vector3(100, 350, 0)

-- 使用这个新对象来设置游戏对象 "player" 的位置。
go.set_position(position, "player")
```

向量的维度也可以超过 3。Defold 使用 `vector4` 对象的四个属性来表示颜色。前三个属性表示颜色的红、绿、蓝，最后一个属性表示半透明程度，也被称为 “Alpha” 值。

在日常生活中，我们习惯用标量值做数学运算、以实数描述数轴上的点等等。我们使用标量来表达更多不同的事物。数字 12 能表达数米的距离、千克、磅、秒、米/秒、伏特或价格。向量也可以表达不同的事物。你已经了解到如何用向量描述对象的位置、颜色，它们也非常适合用来描述对象的空间运动。

要描述计算机屏幕（2维平面）的运动行为，你需要两个值：沿 X 轴的速度与沿 Y 轴的速度。这俩标量值可以很方便的，将速度分别增加到 X、Y 轴上：

```lua
position_x = position_x + speed_x * elapsed_seconds
position_y = position_y + speed_y * elapsed_seconds
```

这约等于你为了让飞船向上移动所做的，像这样的计算行为并没有错，但若使用向量，表述将会更简洁、清晰。由于向量描述了 *方向* 与 *数量*，这更贴合运动的逻辑：方向表明运动方向，数量表明运动速度：

```lua
position = position + speed * elapsed_seconds
```

既然向量用 `position` 和 `speed` 来表现这个空间的坐标，你就能通过加减运算来移动坐标，通过乘除运算来等比例的改变坐标。这些操作是 *向量代数* 的核心部分。

## 向量代数

向量代数定义了向量使用的数学运算。从最简单的逆转（逆转方向）、加法、减法运算来说说吧。

逆转（Negation）
：逆转向量 *v* 就是逆转它的每个属性。所以运算结果是 -*v*，意味着向量相较于运算前的状态，指向了完全相反的方向，且具备一样的数量：

  <img src="doc/vector-neg.png" srcset="doc/vector-neg@2x.png 2x">

加法（Addition）
：将向量 *u* 与向量 *v* 相加。对于 *u* + *v*，我们将 *u* 的每个属性与 *v* 的对应属性相加即可。这将产生一个新的向量：

  <img src="doc/vector-add-cs.png" srcset="doc/vector-add-cs@2x.png 2x">

  通常，从坐标系移动向量的位置，将使操作更易懂：

  <img src="doc/vector-add.png" srcset="doc/vector-add@2x.png 2x">

减法（Subtraction）
：将向量 *v* 减去向量 *u* 的值。对于 *u* - *v*，等价于 *u* 与逆转的向量 *v* 的加法。所以 *u* - *v* = *u* + (-*v*)：

  <img src="doc/vector-sub.png" srcset="doc/vector-sub@2x.png 2x">

用标量相乘（Multiplication with scalar）
：使用实数 *r* 乘以向量 *v* 会产生新的向量，相比旧向量，它等比例的改变了数量值：这个向量由因子 *r* 来延伸或缩小。当因子 *r* 为负数，方向将翻转 180°：
  
  <img src="doc/vector-mul.png" srcset="doc/vector-mul@2x.png 2x">

这些向量基础运算是你一直要用的。此外，有两个特殊操作或许会简化你的运算量，譬如当你需要检查两个向量是否相互平行或成夹角：

点积（Dot product）
：关于向量 *u* 和 *v* 的点积，以 *u ∙ v* 表示，结果是标量值。它被定义为：

  <img src="doc/dot.png" srcset="doc/dot@2x.png 2x">

  - *‖u‖* 是向量 *u* 的数量值；
  - *‖v‖* 是向量 *v* 的数量值；
  - *θ* 是两向量之间的角度。

  <img src="doc/vector-dot.png" srcset="doc/vector-dot@2x.png 2x">

  如果两向量呈直角（两向量之间为 90°），它们的点积为 0。

叉积（Cross product）
：关于向量 *u* 和 *v* 叉积，以 *u* × *v* 表示，结果为垂直于向量 *u* 和 *v* 的新向量（对应的，叉积运算也只在三维向量上生效）：

  <img src="doc/vector-cross.png" srcset="doc/vector-cross@2x.png 2x">

  如果结果是个零向量，这说明：

  - 两个向量中，一个或两个都是零向量（*u* = 0 or *v* = 0）
  - 如果两个向量是平行的 (θ = 0°)
  - 如果两个向量是反平行的 (θ = 180°)

## 使用向量创建运动

使用向量代数，你现在能以更简洁的重写飞船的移动代码。

打开 ["spaceship.script"](defold://open?path=/main/spaceship.script) 并更新 `init()`，`update()` 和 `on_input()` 函数:

```lua
function init(self)
    msg.post(".", "acquire_input_focus")
    self.input = vmath.vector3()                -- [1]
end
```
1. 使用 `vector3` 类创建零向量对象，用于存储用户输入的方向。它被放在了当前的脚本实例（`self`）中，所以它能在飞船游戏对象的整个生命周期中被调用。

```lua
function update(self, dt)
    local movement = self.input * 3             -- [1]
    local p = go.get_position()                 -- [2]
    go.set_position(p + movement)               -- [3]
    self.input = vmath.vector3()                -- [4]
end
```
1. 基于玩家输入的向量，计算移动向量。
2. 获取游戏对象自身的位置（飞船）。这个位置也是个 `vector3` 对象。
3. 设置当前游戏对象的位置为 `p` 加上移动向量。
4. 将 input 向量归零。每一帧中，`on_input()` 函数都在 `update()` 前执行，并执行设置 input 向量的工作。

```lua
function on_input(self, action_id, action)
    if action_id == hash("up") then
        self.input.y = 1                     -- [1]
    elseif action_id == hash("down") then
        self.input.y = -1                    -- [1]
    elseif action_id == hash("left") then
        self.input.x = -1                    -- [1]
    elseif action_id == hash("right") then
        self.input.x = 1                     -- [1]
    elseif action_id == hash("click") and action.pressed then
        print("CLICK!")
    end
end
```
1. 基于玩家的操作，设置 input 向量的 x、y 值。如果玩家同时按下 `up` 和 `left` 键，这个函数将被调用两次并且 x、y 都被设置，input 向量将的运动方向被设置为对角线（斜上方）。

对于这段代码，这会产生两个问题:

首先，如果玩家只是水平或垂直的移动，input 向量的长度为 1，但对角线的长度则是 1.4142（2 的平方根），所以对角线移动会更快。你可能不希望发生这种事。

其次，向量变动的单位是像素/每帧，但没办法确认每帧的时间长度。目前设置为 3 像素的每帧移动速度（于对角线是 4.2 像素/秒)。你可以改变更高的值，使移动速度更快。当然，降低该值就可以移动的更慢。如果你能用像素/秒表示移动速度，这是更好的交流与展现方式。

第一个问题很好解决，只需标准化 input 向量，就可以使它的长度始终为 1：

```lua
function update(self, dt)
    if vmath.length_sqr(self.input) > 1 then        -- [1]
        self.input = vmath.normalize(self.input)
    end
    local movement = self.input * 3
    local p = go.get_position()
    go.set_position(p + movement)
    self.input = vmath.vector3()
end
```
1. 如果向量的平方长度大于 1，将它标准化为 1 即可。比较平方值的长度，这相对于比较长度值更快。

第二个问题需要使用时间步长值。

## 时间步长（Time step）

每一帧，Defold 引擎都会调用所有脚本的 `update()` 函数。一款 Defold 制作的游戏，通常在 60 帧/秒的速度运行，所以一帧有 0.016666 秒那么长——这是再调用 `update()` 的时间间隔。速度向量的数量为 3 意味着 3 * 60 = 180 像素/秒的速度（在正常的渲染脚本中），*如果真的是 60 帧/秒*。如果因为某些原因导致帧率变慢？当前代码带来的移动速度，将是不稳定且不可预料的。

“像素/秒”允许游戏在可变帧率的状态下正常运行，你也能使用秒数来计算游戏中的距离与时间，这是更好的方式。

Defold 的 `update()` 函数支持时间步长作为参数值。这个参数通常为 `dt`（时间增量），它的值是距离上一帧的时间长度，单位是数字秒。如果你对 `dt` 缩放了速度，你将得到合适的单位：

```lua
function update(self, dt)
    if vmath.length_sqr(self.input) > 1 then
        self.input = vmath.normalize(self.input)
    end
    local movement = self.input * 150 * dt              -- [1]  
    local p = go.get_position()
    go.set_position(p + movement)
    self.input = vmath.vector3()
end
```
1. 当前速度为 150 像素/秒。游戏屏幕的宽为 1280 像素，飞船需要 8.53 秒飞完全程。你可以编写个计时器来确认这一点。

[再次运行游戏](defold://build) 并尝试新的移动代码。它应该是正常工作的，尽管动作僵硬，而不是像正常飞行物一样灵动。为飞船赋予真实感的一个好方法是：让玩家控制飞船的加速度，而非移动速度。

## 加速度

上面的代码中，速度被设为固定值，这表示能通过速度与时间步长的相乘，来获得通过时间步长（`dt`）进行的移动速度或平移距离：*移动距离* = *速度* * *dt*，也就是下图中橙色区域：

<img src="doc/integration-constant.png" srcset="doc/integration-constant@2x.png 2x">

设定加速度可以快速地改变速度与方向。加速度通过每帧的时间步长来改变速度值。速度作用于每一帧，所以每帧都会移动到对应的位置。而由于速度随时间变化，移动速度也将像下面的这道曲线产生波动。在数学中，这被称为 [随时间变化的积分](http://en.wikipedia.org/wiki/Integral)。

<img src="doc/integration.png" srcset="doc/integration@2x.png 2x">

在时间步长非常小，通过假设 *v0* 与 *v1* 之间的加速度是恒定的，我们可以有效的计算出该区域的几何近似值，这意味着速度在两点之间线性波动。根据这样的假设，*v1* 能通过 *v0* + *加速度* * *dt* 的公式来计算，运动结果为：

<img src="doc/movement.png" srcset="doc/movement@2x.png 2x">

现在，你能写下 `init()` 与 `update()` 函数的最终代码了（`on_input` 保持不变哟）：

```lua
function init(self)
    msg.post(".", "acquire_input_focus")
    self.velocity = vmath.vector3()             -- [1]
    self.input = vmath.vector3()
end

function update(self, dt)
    if vmath.length_sqr(self.input) > 1 then
        self.input = vmath.normalize(self.input)
    end
    
    local acceleration = self.input * 200       -- [2]
    
    local dv = acceleration * dt                -- [3]
    local v0 = self.velocity                    -- [4]
    local v1 = self.velocity + dv               -- [5]
    local movement = (v0 + v1) * dt * 0.5       -- [6]

    local p = go.get_position()
    go.set_position(p + movement)               -- [7]

    self.velocity = v1                          -- [8]
    self.input = vmath.vector3()
end
```
1. 创建一个向量来存储随时间变化的速度；
2. 加速度设为在特定方向变动 200 像素/秒；
3. 计算在当前时间步长上的速度变化；
4. v0 是上一帧的速度；
5. v1 是 v0 加上这一帧的速度变动；
6. 计算飞船要在这一帧的移动多少距离；
7. 改变游戏对象的当前位置；
8. 存储 v1 速度值，在下一帧时继续使用它。

现在，是时候去 [获取能转起来的新重型飞船](defold://build) 了。

恭喜你！你完成了这个教程。但别停下，继续去写更有趣的代码吧。

这里有些想法，你可以尝试一下：

1. 通过向量设定速度上限；
2. 让飞船无法飞出屏幕边缘，从而留在显示屏内；
3. 允许通过鼠标点击来决定前行方向。

通过 [文档](https://defold.com/learn) 获取更多案例、教程、手册以及 API 文档。

如果你遇到了困难，可以来我们的 [论坛](https://forum.defold.com) 寻求帮助。

Happy Defolding!

----

本项目遵循 Creative Commons CC0 1.0 Universal 许可发布。

你可以在任何项目中自由的使用这些资源，无论个人或商业用途。无需通过询问我们来获得授权。表示资源的来源不是必须的，但如果你这样做了，很感谢你为我们署名！
[阅读完整的许可协议](https://creativecommons.org/publicdomain/zero/1.0)。
