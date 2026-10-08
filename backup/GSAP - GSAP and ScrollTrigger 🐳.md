```js
gsap.registerPlugin(ScrollTrigger)
```
- registerPlugin(ScrollTrigger)  GSAP开始调用 ScrollTrigger 。ScrollTrigger 允许基于滚动事件创建各种响应式动画，如当用户滚动到某个位置时开始/暂停/反转动画、根据滚动位置动态改变动画属性、实现视差滚动效果、触发事件等。通过注册此插件，后续使用GSAP创建动画时可以利用ScrollTrigger提供的方法和功能。

```js
gsap.to('.title_paralax', {
    scrollTrigger: {
        trigger: '.header',
        start: 'top top',
        scrub: 1.9,
    },
    yPercent: -150,
});
```
- .title_paralax 这里指定了要应用动画的元素，即选择器 .title_paralax 对应的 HTML 元素
- gsap.to() method gsap.to() 是 GSAP 中用于创建目标元素从当前状态过渡到指定状态的动画方法
- ScrollTrigger configuration {} 内的 scrollTrigger 对象定义了与滚动事件相关的动画触发条件和行为
  - trigger: .header 指定触发动画的滚动触发器元素，即当 .header 元素在视口中达到特定位置时，动画开始或改变状态
  - start: 'top top' 设置动画开始的滚动位置。在这个例子中，"top top" 表示当触发器（.header）到达窗口顶部（垂直方向）且触发器顶部边缘与视口顶部对齐时，动画开始
  - scrub: 1.9 设置滚动 scrubbing（擦除）行为。这是一个数值（大于0且小于等于2），表示动画与滚动速度的关系。值为 1.9 意味着动画速度是滚动速度的1.9倍。当用户快速滚动时，动画会以更快的速度进行；反之，缓慢滚动则动画速度较慢。这种效果可以增强视觉上的动态关联性和沉浸感
- Animation properties 在 scrollTrigger 对象之外，还定义了具体的动画属性：
  - yPercent: -150 这个属性控制 .title_paralax 元素在垂直方向上相对于其自身高度的位置变化。值 -150 表示当动画触发时，该元素将向上偏移自身高度的150%。随着页面滚动，根据 scrollTrigger 的设置，这个偏移量将随滚动速度的变化而动态调整
> 当页面滚动，.header 元素到达窗口顶部时，开始对 .title_paralax 元素应用一个垂直方向上的位移动画。该动画使其向上偏移自身高度的150%，且动画速度与滚动速度成1.9倍关系（即滚动 scrubbing 效果）

```js
const gsapSq = gsap.utils.toArray('.section-title__square'); 

gsapSq.forEach((gSq, i) => { 
    const rotat = gsap.from(gSq, 3, { rotation: 720 }); 
    ScrollTrigger.create({ trigger: gSq, animation: rotat, start: 'top bottom', scrub: 1.9, }); 
});
```
- ` const gsapSq = gsap.utils.toArray('.section-title__square'); ` 这行代码利用GSAP提供的utils.toArray()方法，将类名为'.section-title__square'的所有DOM元素收集到一个数组gsapSq中
- ` gsapSq.forEach((gSq, i) => { ... }); ` 遍历上述数组中的每一个元素gSq，并给每个元素执行一个匿名函数。这里的i是元素在数组中的索引
- ` const rotat = gsap.from(gSq, 3, { rotation: 720 });  `对于数组中的每个元素gSq，使用gsap.from()方法创建一个初始动画。动画持续时间为3秒，作用是让元素旋转720度。这意味着当元素进入视口时（或者动画开始时，取决于ScrollTrigger配置），它会从旋转720度的状态逐渐恢复到初始状态（通常是0度）
- ` ScrollTrigger.create({ ... }); ` 对于每个.section-title__square元素，创建一个新的ScrollTrigger实例，并配置相应的滚动触发动画
  - trigger: gSq 触发器设置为当前遍历到的元素gSq，这意味着当这个元素滚动到视口的特定位置时，动画将会开始或更新。 
  - animation: rotat 指定与触发器关联的动画就是之前通过gsap.from()创建的旋转动画
  - start: 'top bottom' 动画开始的触发点设置为当触发器元素滚动到视口底部时。这意味着当.section-title__square元素完全出现在视口底部时，动画启动
  - scrub: 1.9 同样设置了滚动 scrubbing 参数为1.9，意味着动画播放的速度与滚动速度相关联，跟随滚动的速度按1.9倍速率变化
 > 这段代码会对所有类名为.section-title__square的元素分别创建一个初始旋转动画，并通过ScrollTrigger与滚动事件绑定。当这些元素滚动至视口底部时，它们将以1.9倍滚动速度的对应速率开始从720度旋转回原位

```js
const gTl = gsap.timeline(); 

gTl.from('.title .char', 1, { opacity: 0, yPercent: 130, stagger: 0.06, ease: 'back.out' });
gTl.to('.header__img', 2, { clipPath: 'polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)', scale: 1, ease: 'expo.out' }, '-=1');
gTl.from('.header__marq', 2, { opacity: 0, yPercent: 100, ease: 'expo.out' }, '-=1.5');
```
- ` const gTl = gsap.timeline(); ` 创建一个空的GSAP时间轴实例，命名为gTl。这个时间轴将用于组织和同步多个动画。
- ` gTl.from('.title .char', 1, { opacity: 0, yPercent: 130, stagger: 0.06, ease: 'back.out' });  `在gTl时间轴上添加一个“from”动画序列：
  - 目标选择器为.title .char，即.title元素内部所有具有.char类的子元素
  - 动画持续时间为1秒
  - 动画属性：
    - opacity: 0: 将目标元素的透明度从0渐变到其原始值（即完全可见）
    - yPercent: 130: 将目标元素垂直向下偏移130%，然后回到其原始位置
    - stagger: 0.06: 为所有选中的元素应用延迟动画，使得每个元素的动画开始时间依次相差0.06秒，形成错落有致的效果
    - ease: 'back.out': 使用“back.out”缓动函数，为动画赋予一种类似弹簧拉伸后回弹的运动感觉
- ` gTl.to('.header__img', 2, { clipPath: 'polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)', scale: 1, ease: 'expo.out' }, '-=1'); ` 在gTl时间轴上添加一个“to”动画序列：
  - 目标选择器为.header__img
  - 动画持续时间为2秒
  - 动画属性：
    - ` clipPath: 'polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%)' ` : 将目标元素的剪切路径设置为一个完整的矩形，这通常用来取消之前的剪切效果，使其完整显示
    - ` scale: 1 ` : 将目标元素的缩放比例从其当前值渐变到1（即无缩放）。如果元素原本未缩放，则保持不变；如果已缩放，则恢复到原始尺寸
    - ` ease: 'expo.out' ` : 使用“expo.out”缓动函数，为动画赋予一种缓慢加速然后快速减速的运动感觉
  - 参数'-=1'表示该动画相对于前一个动画（.title .char的动画）延迟1秒开始
- ` gTl.from('.header__marq', 2, { opacity: 0, yPercent: 100, ease: 'expo.out' }, '-=1.5');  ` 在gTl时间轴上再添加一个“from”动画序列：
  - 目标选择器为.header__marq
  - 动画持续时间为2秒
  - 动画属性与第2个动画类似，但没有clipPath属性：
    - opacity: 0: 将目标元素的透明度从0渐变到其原始值（即完全可见）
    - yPercent: 100: 将目标元素垂直向下偏移100%，然后回到其原始位置
    - ease: 'expo.out': 依然使用“expo.out”缓动函数
  - 参数'-=1.5'表示该动画相对于前一个动画（.header__img的动画）延迟1.5秒开始。
> 这段代码创建了一个GSAP时间轴gTl，包含了三个依次触发的动画序列。第一个序列使.title .char元素依次淡入并从下方滑入；第二个序列在第一个序列结束后1秒，使.header__img元素恢复完整显示并可能调整其缩放至原始尺寸；第三个序列在第二个序列结束后1.5秒，使.header__marq元素依次淡入并从下方滑入。所有动画均使用特定的缓动函数来赋予自然的运动效果