ES5中Array新增的9个API

ES5规范在Array的原型上新增了9个方法，分别是forEach、filter、map、reduce、reduceRight、some、every、indexOf 和 lastIndexOf

1.forEach(callback[,thisArg]);

在ES5之前，我们可以通过for和for in 两种方式来遍历数组，而ES5引入了一个新方法forEach，使数组遍历更加简洁，forEach需要传递2个参数，第一个参数是回调函数，是必选参数，第二个参数是一个对象，用来改变callback中的this指向，是可选参数。

