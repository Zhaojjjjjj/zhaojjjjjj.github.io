> Note: When the names of two switching components are the same, a key attribute needs to be added to distinguish the two components, otherwise the animation will not take effect
```html
<div>
    <transition name="fade">
        <p key=1 v-if="show">hello</p>
        <p key=2 v-else>hi</p>
    </transition>
</div>
```
```css
.fade-enter-active,
.fade-leave-active {
    transition: opacity .5s
}

.fade-enter,
.fade-leave-to {
    opacity: 0
}
```