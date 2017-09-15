---
title:  "What is Polymorphism"
date:   2017-09-15 14:08 UTC
tags:
---

Polymorphism, in Greek, literally means “many forms”, but what on earth does that mean in regards to computer software and why should we care?

Polymorphism is one of the 4 basic principles of Object Oriented Programming and makes our code extremely flexible and modular.

>Polymorphism means that the sender of a stimulus does not need to know the receiving instance’s class. The receiving instance can belong to an arbitrary class.

Ivar Jackobson, Object-Oriented Software Engineering – Page 55

What this means is that the function, class method etc. that sends the stimulus doesn’t care the exact concrete class it is calling just that it can respond to a particular method.

Let’s look at a use case to build a form with an Input and TextArea to explain this point.

First, we define a FormElement interface to represent each form element and to ensure that a contract is defined and that polymorphism is made possible.

```php
interface FormElement {
   public function render();
}
```

Now, lets create our concrete form element classes for an Input and a TextArea. You can ignore the invalidity or lack of names, values etc. for now as this is not a fundamental part of this principle and is such irrelevant here.

```php
class Input implements FormElement {
   public function render() {
      return '<input type="text" />';
   }
}
```
	
```php
class TextArea implements FormElement {
   public function render() {
      return '<textarea></textarea>';
   }
}
```

Now we can create a form class that acts as a form component collection, adding form components as we go which finally renders all our form elements and could look like the below:

```php
class Form {
    
   protected $components = [];
    
   public function addComponent(FormElement $component) {
      $this->components[] = $component;
   }
 
}
```

Nothing special about the class above, the Form class allows us to add any component that implements the FormComponent interface which means we know for sure that each component will have a render() function and as such we can use polymorphism to render our form elements, the polymorphic part is highlighted below:

```php	
class Form {
    
   protected $components = [];
    
   public function addComponent(FormElement $component) {
      $this->components[] = $component;
   }
 
   public function render() {
      foreach($this->components as $component)
         echo $component->render();
   }
 
}
```

As you can see above, the calling code (the Form class) doesn’t care what component is used (Input, TextArea, Select, Submit, Button etc.), it just knows it will have a render() method available to use.

The code can then be built and called like this:

```php
$form = new Form;
$form->addComponent(new Input);
$form->addComponent(new TextArea);
$form->render();
```

There are a lot of nice side effects of this, as it means we don’t need to modify our calling code when we add more classes to deal with this. If you have switch statements in your code it’s usually a sign that you need to refactor to polymorphism, for example the below shows how the above may be built without polymorphism (again, highlighted below):

```php	
class Form {
    
   protected $components = [];
    
   public function addComponent($component) {
      $this->components[] = $component;
   }
 
   public function render() {
      foreach($this->components as $component):
         switch(get_class($component)):
            case 'Input' : $component->displayInput(); break;
            case 'TextArea' : $component->displayTextArea(); break;
         endswitch;
      endforeach;
   }
 
}
```

This also now breaks the Open/Closed Principle as if we add a new form component we will need to open up this class and add a new line to our case statement, whereas before we could add new elements at ease.

There are many other types of Polymorphism but I believe this is a good, practical overview to get you started.