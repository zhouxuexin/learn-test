ES5中Array新增的9个API

ES5规范在Array的原型上新增了9个方法，分别是forEach、filter、map、reduce、reduceRight、some、every、indexOf 和 lastIndexOf

1.forEach(callback[,thisArg]);

在ES5之前，我们可以通过for和for in 两种方式来遍历数组，而ES5引入了一个新方法forEach，使数组遍历更加简洁，forEach需要传递2个参数，第一个参数是回调函数，是必选参数，第二个参数是一个对象，用来改变callback中的this指向，是可选参数。

var arr=['a','b','c'];
	arr.forEach(function(v,i,r){
		console.log(v,i,r);//==>a 0 ["a", "b", "c"]
				     // b 1 ["a", "b", "c"]
				     // c 2 ["a", "b", "c"]

从输出的接口可以看出，callback中传入了3个参数v,i,r 分别表示当前元素、当前位置、数组对象。再看看使用thisArg的例子：

var obj = {
   print:function(a,b){
       console.log(a,b);
   }
};
var arr = ['a','b','c'];
arr.forEach(function(v,i,a){
   this.print(v,i);
},obj);

不传thisArgs时，callback中的 this 默认指向window对象，当传递thisArg时，callback中的this就指向了thisArg,因此这个参数的目的就是为了改变回调函数中的this指向。

对于不支持ES5的浏览器，我们可以对forEach进行简单的扩展来兼容老的浏览器：

if(!Array.prototype.forEach){
    Array.prototype.forEach = function(callback,thisArg){
        for (var i=0;i<this.length;i++){
            //当thisArg为undefined时，JS引擎会将window作为其调用者
            callback.call(thisArg,this[i],i,this.toString());
        }
    }
}
filter(callback[,thisArg])

filter是`过滤`的意思，所以这个方法的作用就是返回一个匹配过滤条件的新数组，其接收两个参数callback和thisArg, callback也是回调函数，主要用于对元素进行条件匹配，thisArg和forEach中的thisArg作用一样，在这里就不重复了，看下面示例：

var arr = ["a","b","a","c"];
var newArr = arr.filter(function(item){
     return item === "a";
});

newArr -> ["a","a"]

代码很简单，一看就明白，没有filter的时候，要实现这个功能，我们事先要创建一个空的数组，把匹配到的元素再push进去，现在就不需要那么麻烦了，我们再看看对filter的扩展：

if(!Array.prototype.filter) {
    Array.prototype.filter = function (callback, thisArg) {
        var temp = [];
        for (var i = 0; i < this.length; i++) {
           if(callback.call(thisArg,this[i])){
               //如果callback返回true,则该元素符合过滤条件，将元素压入temp中
               temp.push(this[i]);
           }
        }
        return temp;
    }
}
map(callback[,thisArg])

map的作用是对原数组进行加工处理后并将其作为一个新数组返回，该方法同样接收两个参数，callback是回调函数用于对数组进行加工处理，thisArg和上面的一样。先看一个简单的例子你就明白了：

var arr = [
   {w:10,h:10}, //定义长和宽
   {w:15,h:20},
   {w:12,h:12}
];

var newArr = arr.map(function(item){
   //根据长宽计算出面积并赋值给新属性area
   item.area = item.w * item.h;
   return item;
});

newArr[0] - > {w: 10, h: 10, area: 100}

可以看出，newArr返回的是增加了area属性的对象数组。这个方法非常实用，一般情况下，当一个ajax请求返回时，我们都要对其结果集进行过滤和校验等操作，这时map就派上用场了。我们再看看如果对map进行兼容性扩展：

if(!Array.prototype.map) {
   Array.prototype.map = function (callback, thisArg) {
       var temp = [];
       for (var i = 0; i < this.length; i++) {
           var newItem = callback.call(thisArg,this[i]);
           temp.push(newItem); //将callback返回的新元素压入temp中
       }
       return temp;
   }
}

reduce(callback[,initialValue])

reduce在这里有`减少`的意思，那reduce到底是干什么用的呢？说实话我也不太理解，看看比较官方的解释：The method  applies  a  function  against  an accumulator and each  value  of the array (from  left -to- right ) to reduce it to  a  single  value . 自己慢慢的理解吧，我们先看看怎么使用吧，用的多了自然就明白了：

var arr = [1,2,3,4];
var newArr = arr.reduce(function(previousValue, currentValue, currentIndex, array){
    console.log(previousValue, currentValue,currentIndex);
    return previousValue + currentValue;
});

1 2 1
3 3 2
6 4 3

newArr -> 10

从运行结果可以看出，reduce实现了数组元素的累加，reduce接收4个参数，previousValue中存放的是上一次callback返回的结果，currentValue是当前元素，currentIndex是当前元素位置，array是当前数组。previousValue初始值为数组的第一个元素，数组从第2个元素开始遍历。我们再来看看initialValue参数究竟是什么鬼：

var arr = [1,2,3,4];
var newArr = arr.reduce(function(previousValue, currentValue, currentIndex, array){
    console.log(previousValue, currentValue,currentIndex);
    return previousValue + currentValue;
},100);

100 1 0
101 2 1
103 3 2
106 4 3

newArr -> 110

从运行结果看，initialValue参数指定了previousValue的初始值，更重要的是，这次数组是从第1个位置开始遍历，而不再是从第2个位置开始了。 现在回过头来，对照这两个例子，我相信你一定能够理解reduce的作用了。下面对于reduce的扩展会巩固你对reduce的理解：

if(!Array.prototype.reduce) {
   Array.prototype.reduce = function (callback, initialValue) {
        var previousValue = initialValue || this[0];//如果不指定intialValue,则默认为数组的第一个元素
        //如果不指定initialValue，i从1开始遍历，否则就从0开始遍历
        for (var i = initialValue?0:1; i < this.length; i++) {
            //previousValue 累加每一次返回的结果
            previousValue += callback(previousValue, this[i],i,this.toString());
        }
        return previousValue;
    }
}
reduceRight(callback[,initialValue])

reduce的作用完全相同，唯一的不同是，reduceRight是从右至左遍历数组的元素。

some(callback[,thisArg])

some是`某些、一些`的意思，因此，some的作用是检测数组中的每一个元素，当callback返回true时就停止遍历，并返回true，这样的描述似乎有些抽象，看代码，一切尽在代码中：

var arr = [ 1, 2, 3, 4];
var result = arr.some( function( item, index, array ){
    console.log( item, index, array);
    return item > 2;
});
->
 1 0 [1, 2, 3, 4]
 2 1 [1, 2, 3, 4]
 3 2 [1, 2, 3, 4]

 restule -> true

从运行结果看，some检测整个数组，只要当arr中有一个元素符合条件item>2 就停止检测和遍历，并返回true，以表示检测到目标。这和我们在for循环中使用break语言的作用有点类似，这会儿你应该明白some的作用了吧！ 下面对于some的扩展会有助于你对some的理解：

if(!Array.prototype.some) {
   Array.prototype.some = function (callback, thisArg) {
        for (var i = 0; i < this.length; i++) {
           if(callback.call(thisArg,this[i],i,this.toString())){

               return true; //检测到callback返回true,跳出循环，并返回true
           }
        }
        return false; //一个符合条件的都没有检测到，返回false
    }
}
every(callback[,thisArg])

every是`每一个`的意思，相比some来讲，every对元素的检测应该更加严格，那every到底是干什么的呢，看代码就知道了：

var arr = [ 1, 2, 3, 4];
var result = arr.every( function( item, index, array ){
    console.log( item, index, array );
    return item < 3;
});

 1 0 [1, 2, 3, 4]
 2 1 [1, 2, 3, 4]
 3 2 [1, 2, 3, 4]

 result -> false

从运行结果看，当检测第3个元素时，item<2为false, 停止检测，并返回false, 这说明every在检测元素时，要求每一个元素都要符合条件item<3,如果有一个不符合就停止检测，并返回false,(ps：你可以测试item<5时的运行结果，返回值一定是true). 那every到底有什么用武之地呢？ 当一个for循环使用了break语句后，我们想知道for循环是否正常的执行完时， 我们一般会通过检测for中的索引i==arr.length来判断,因此every的作用就体现在这里。 我们再看看对于every的扩展：

if(!Array.prototype.every) {
   Array.prototype.every = function (callback, thisArg) {
        for (var i = 0; i < this.length; i++) {
           if(!callback.call(thisArg,this[i],i,this.toString())){

               return false; //检测到不符合条件的元素,跳出循环，并返回false
           }
        }
        return true; //所有元素都符合条件，返回true
    }
}
indexOf 和 lastIndexOf

这两个方法和String类中indexOf和lastIndexOf作用类似，相信大家对这两个方法用的很熟了，因此这里不多做解释了。
