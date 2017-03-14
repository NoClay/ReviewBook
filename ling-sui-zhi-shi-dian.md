# Activity中的琐碎知识点

布局中的每一个View默认实现了onSaveInstanceState\(\)方法，这样的话，这个UI的任何改变都会自动地存储和在activity重新创建的时候自动地恢复。**但是这种情况只有在你为这个UI提供了唯一的ID之后才起作用，如果没有提供ID，app将不会存储它的状态。**





