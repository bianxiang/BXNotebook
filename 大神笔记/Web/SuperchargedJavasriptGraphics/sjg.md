[toc]

## 前言

Canvas、WebGL。

本书主题：

- Reusing and optimizing code, including inheritance techniques and performance tips
- Taking advantage of the surprising graphics power of regular DOM manipulation (DHTML)
- 使用 Canvas
- 开发视频游戏
- 图形和动画的数学
- Presenting your data in creative ways with the Google Visualizations API and Google Chart Tools
- 高效使用jQuery；开发jQuery图形插件
- 使用jQuery开发图形化的手机Web App
- Using PhoneGap to create native Android applications from your web applications

## 1. 代码重用与优化

### 原型继承与函数式继承

原型继承的例子：

    var Pet = function (name, legs) {
        this.name = name; // Save the name and legs values.
        this.legs = legs;
    };
    Pet.prototype.getDetails = function () {
    	return this.name + ' has ' + this.legs + ' legs';
    };
    // Define a Cat object, inheriting from Pet.
    var Cat = function (name) {
    	Pet.call(this, name, 4); // 调用父对象的构造器
    };
    // 继承：注意new关键字！
    Cat.prototype = new Pet();
    Cat.prototype.action = function () {
    return 'Catch a bird';
    };
    // Create an instance of Cat in petCat.
    var petCat = new Cat('Felix');
    var details = petCat.getDetails(); // 'Felix has 4 legs'.
    var action = petCat.action(); // 'Catch a bird'.
    petCat.name = 'Sylvester'; // Change petCat's name.
    petCat.legs = 7; // Change petCat's number of legs!!!
    details = petCat.getDetails(); // 'Sylvester has 7 legs'.

原型继承的问题：字段没有可见性保护（如`legs`字段）。

函数式继承的例子：

    var pet = function (name, legs) {
        // 现在legs变成了私有字段
        var that = {
            name: name,
            getDetails: function () {
                return that.name + ' has ' + legs + ' legs';
            }
        };
        return that;
    };
    // Define a cat object, inheriting from pet.
    var cat = function (name) {
        var that = pet(name, 4); // Inherit from pet.
        // Augment cat with an action method.
        that.action = function () {
        	return 'Catch a bird';
        };
        return that;
    };
    // Create an instance of cat in petCat2.
    var petCat2 = cat('Felix');
    details = petCat2.getDetails(); // 'Felix has 4 legs'.
    action = petCat2.action(); // 'Catch a bird'.
    petCat2.name = 'Sylvester'; // We can change the name.
    petCat2.legs = 7; // But not the number of legs!
    details = petCat2.getDetails(); // 'Sylvester has 4 legs'.

现在`legs`变成了私有字段。`petCat2.legs = 7;`实际无法修改它。

原型继承相比于函数式继承，存储上更节省空间：对象从原型继承的属性和方法只会被存储一次（在最初的原型那里）。

## 2. DHTML基础

















