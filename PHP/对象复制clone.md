```php
# 在Laravel下同一方法下复制Model对象分别进行操作无效
public function demo(Request $request){
    # 方法 1 无效
	$model = Model::where('id',$id);
    $a = $model->where('aid',$aid);
    $b = $model->where('bid',$bid);
    
    # 方法 2 无效
    $modelA = Model::where('id',$id);
    $modelB = $modelA;
    $a = $modelA->where('aid',$aid);
    $b = $modelB->where('bid',$bid);
    
   # 方法 3 有效
    $modelA = Model::where('id',$id);
    $modelB = clone $modelA;
    $a = $modelA->where('aid',$aid);
    $b = $modelB->where('bid',$bid);
}
```



在多数情况下，我们并不需要完全复制一个对象来获得其中属性。但有一个情况下确实需要：如果你有一个 GTK 窗口对象，该对象持有窗口相关的资源。你可能会想复制一个新的窗口，保持所有属性与原来的窗口相同，但必须是一个新的对象（因为如果不是新的对象，那么一个窗口中的改变就会影响到另一个窗口）。还有一种情况：如果对象 A 中保存着对象 B 的引用，当你复制对象 A 时，你想其中使用的对象不再是对象 B 而是 B 的一个副本，那么你必须得到对象 A 的一个副本。