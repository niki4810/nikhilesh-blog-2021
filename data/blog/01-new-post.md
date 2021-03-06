---
title: "Data-Binding your Backbone Models to Views"
summary: "One of the first things any developer migrating into the Javascript world would look for is to find common coding practices and concepts"
date: '2013-03-02 15:50'
tags: ["Backbone", "Backbone-ModelBinder", "Data-Binding"]
---

# Prologue

One of the first things any developer migrating into the Javascript world would look for is to find common coding practices and concepts. `Data-Binding` is one such concept which facilitates bi-directional binding between views and its associated models. While each programming language might have its own way for achieving this, Javascript does provide a wide variety of options for implementing `Data-Binding`. In this post, I will show, how easy it is to bind your Backbone models to views using the Backbone.ModelBinder plugin. 

# How can we achieve `Data-Binding` using ModelBinder?

To illustrate this, we will be building a simple application which looks like this: [Link](http://jsfiddle.net/niki4810/CSyAz/embedded/result/). 

* The annotate source code for this example application can be found at: [Source](http://niki4810.github.io/annotate-sources/model-binding.html)
* The complete fiddle can be found at : [Fiddle](http://jsfiddle.net/niki4810/CSyAz/)

The application contains two views, an `editor view` for entering information and a `viewer view` for previewing the same information in read only mode. Both these views display the following information :

* First Name
* Last Name
* Salary in USD
* Professional
* Favorite Search Engine

 
Some of the requirements for our application are

1. As we update a field in the edit view, the corresponding field on the preview view should be updated
2. When we update the `salary` field, the view should format the value as money
3. When the `favorite search engine` field is changed in the edit view, the preview view should render a hyperlink and should update both the `label` and the `href` properties on that hyperlink.


Now that we have our requirements set, as a first step we create a `Backbone Model` and set some default values in it.

```javascript
    //create a instance of Backbone Model with some default values.
    var BaseViewModel = new Backbone.Model();
    BaseViewModel.set({
            "firstName": "",
            "lastName": "",
            "salary": "",
            "pro": true
    });
```

Create a converter function , that formats the given value as money, for example `123` gets converted to `$123.00`. This function takes two parameters: a `value` parameter, which is the amount that needs to be formatted & `direction`, which has two possible values `ModelToView` or `ViewToModel`. Both these values are automatically supplied by the model binder plugin when a change occurs on the binded element.

```javascript
var salaryConverter = function (direction, value) {
    if (direction === "ModelToView") {
        //format only when the direction is from model to view
        return accounting.formatMoney(value);
    } else {
        //from view to model, just store the plain value
        return value;
    }
};
```

For each view we pass in a bindings object, which determines, which element should be bound to which property on the model. For example, to bind the `firstName` property on the model to an element with `name set to firstName` on the edit view, the bindings object would look like:  

```javascript
var editorBinding = {
	"firstName": '[name = "firstName"]'
}
```

The literal value `firstName` points to an `attribute` on the Backbone model, and the `[name="firstName"]` acts a selector for selecting an element in the edit view (which is a Backbone View instance).

### Using the `converter` function

Similar to firstName, we set other bindings for other attributes as well

```javascript
var editorViewBindings = {
        "firstName": '[name = "firstName"]', 
        "lastName": '[name = "lastName"]',
        "salary": {
        selector: '[name = "salary"]',
        converter: salaryConverter
    },
        "pro": '[name = "pro"]',
        "favSearch": '[name = "favSearch"]'
};
```

As you might have already noticed, for the `salary` attribute we set the value as a object which internally has two properties:

* `selector` property determines which element this property should be bound to. In this case an element with `name property set to salary`.

* We assign the `salaryConverter` function to the `converter` property. So whenever there is a change event on element with `[name = "salary"]`, the Backbone ModelBinder plugin calls the `salaryConverter` twice. Once from `Model to view` and the other from `View to Model`. Through this we can achieve our second requirement where we want to format the salary attribute as money when displaying on the view.


### using the `elAttribute`

Similar to the edit view bindings we construct a binding object for the viewer view. The code looks something like this

```javascript
var viewerBindings = {
    "firstName": '[name = "firstName"]',
        "lastName": '[name = "lastName"]',
        "salary": {
        selector: '[name = "salary"]',
        converter: salaryConverter
    },
        "pro": '[name = "pro"]',
        "favSearch": [{
        selector: '[name = "favSearch"]',
        elAttribute: "href"
    }, {
        selector: '[name = "favSearch"]'
    }]
}
```

One major change in this binding object is that, we assign an array to the `favSearch` property. We do this to achieve our third requirement, where we want to bind the label and the href properties on the hyperlink that gets rendered in the viewer view.

While the second element in the array is a simple selector `selector: '[name = "favSearch"]'` the first element introduces us to a new property called `elAttribute`. What this means is that, when ever there is a change on the `favSearch` property, update the `href` property on the element with name `[name = "favSearch"]`. 

We did not specify the `elAttribute` for other bindings because, the ModelBinder applies it on the text property of each element by default.


### Applying our binding

Now that we have our binding objects ready, we will have to create a Backbone view for applying this binding. Since the app we are developing is pretty simple and there is no much difference between both our view, we construct on single backbone view that takes the template id and bindings as parameters. Our view code looks like this

```javascript
//create  a Backbone view
   var BaseView = Backbone.View.extend({
       //local variable for model binder
       _modelBinder: undefined,
       initialize: function () {
           //on view initialize, initialize _modelBinder
           this._modelBinder = new Backbone.ModelBinder();
       },
       close: function () {
           //when view closes, unbind Model bindings
           this._modelBinder.unbind();
       },
       render: function () {
           //when the view is rendered
           //get the templates id from passed in options
           //NOTE: templateId is not a property of Backbone or       ModelBinder, its a custom parameter that we pass into view's constructor
          
           var templateId = "#" + this.options.templateId;

           //construct the template
           var template = _.template($(templateId).html());
           var templateHTML = template();
           //append it to current view
           this.$el.html(templateHTML);
           
           //get the bindings attribute from passed options
            //NOTE: bindings is not a property of Backbone, its a custom parameter that we pass into view's constructor
           var bindings = this.options.bindings;

           //call modelBinder bind api to apply bindings on the current view
           this._modelBinder.bind(
           this.model /*the model to bind*/ ,
           this.el /*root element*/ ,
           bindings /*bindings*/ );

           return this;
       }
   });
```

While the above code simply shows a standard Backbone view object. There are two main parts to focus. In the `initialize` function we create a new instance of model binder and assign it to a local variable.

```javascript
 this._modelBinder = new Backbone.ModelBinder();
```

The main place where our bindings are applied is in the `render` function. 

```javascript
this._modelBinder.bind(
this.model /*the model to bind*/ ,
this.el /*root element*/ ,
bindings /*bindings*/ );
```

As we can see we simply call the `bind` function on the modelBinder to bind a model to the view's root element. This function accepts three parameters `the model` a backbone model containing attributes information, `root el` the parent level element of the backbone view and `bindings` parameter which specific the relationship between the backbone model attributes and view elements.

Now that we have everything setup all we need to do is render both the views and append it to the DOM.

```javascript
//instantiate the editor view by passing the model, template id and 
//bindings into the constructor
var myEditorView = new BaseView({
    model: BaseViewModel,
    templateId: "editor-template",
    bindings: editorViewBindings
});

 //instantiate the viewer view by passing the model, template id and 
//bindings into the constructor
var myViewerView = new BaseView({
    model: BaseViewModel,
    templateId: "viewer-template",
    bindings: viewerBindings
});

//append both the Backbone views to the container
$(".container").append(myEditorView.render().$el);
$(".container").append(myViewerView.render().$el);
```

As mentioned earlier, the complete source code for this example can be found at the following [fiddle](http://jsfiddle.net/niki4810/CSyAz/). Although this looks like a lot of code, it simple once we understand the basic concept behind model binding.


# What are other alternatives for `Data-Binding` Backbone Views?

As they say, there are a trillion ways for doing million things, Here are some other options for `Data-Binding` your Backbone Models to Views.

* [KnockBack](http://kmalakoff.github.io/knockback/)
* [Rivets.js](http://rivetsjs.com/)

I personally feel ModelBinder is much simpler in terms of implementation than the above two frameworks.


# About Backbone.ModelBinder

Special thanks to [Bart Wood](https://github.com/theironcook) for developing such an awesome framework. More details can be found at [Backbone.ModelBinder](https://github.com/theironcook/Backbone.ModelBinder).


