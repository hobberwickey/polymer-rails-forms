polymer-rails-forms
===================
##What this does

Polymer-rails-forms give you an extendable polymer element that let's you create complex 
activerecord compatible forms in polymer by simply defining a form's structure and 
optionally data via javascript. Check out the <a href='http://deadlykitten.com/polymer-rails-forms'>demo page</a>
to see it in action. 

##Installation

first follow the installation instructions for emcee

```
	bower install --save polymer-rails-form
```

then import it via 

```
<link rel='import' href='/app/wherever/bower_components/polymer-rails-forms/rails-form.html'>
```

##Basic example

This is what a simple login form could look like using this gem

```html
<rails-form id='sign_in_form' action="/users/sign_in" method="POST" scope="user" submitText="Sign In"></rails-form>
<script>
	document.getElementById("sign_in_form").setAttribute("structure", JSON.stringify([
		  { key: "email", type: 'string', label: "Email Address" },
    	{ key: "password", type: 'password' }
    ]))
</script>
```

This syntax, however short is a bit clunky, a better way of using this gem is to create forms by creating a 
polymer element that extends rails-form. Here's the same login form


```html
<link rel="import" href="../rails-forms/rails-form.html" >
<polymer-element name="login-form" extends='rails-form'>
  <shadow></shadow>
  
  <script>
    Polymer({
      action: "/users/sign_in",
      method: "POST",
      scope: "user",
      submitText: "Sign In",

      ready: function(){
        this.structure = [
          { key: "email", type: 'string', label: "Email Address" },
          { key: "password", type: 'password' },
        }               
      ]
    });
  </script>
</polymer-element>
```

This syntax gives you the flexibility to add encapulated methods and styles. It's a lot 
better for complex forms. 

Nested attributes are supported (otherwise what's the point right?). This is that login form 
again, but this time with nested location attributes

```html
<link rel="import" href="../rails-forms/rails-form.html" >
<polymer-element name="login-form" extends='rails-form'>
  <shadow></shadow>
  
  <script>
    Polymer({
      action: "/users/sign_in",
      method: "POST",
      scope: "user",
      submitText: "Sign In",

      ready: function(){
        this.structure = [
          { key: "email", type: 'string', label: "Email Address", required: true },
          { key: "password", type: 'password', required: true },
          { key: "location", type: 'nest', allowAdd: false, multiple: false, structure: [
            { key: "address", type: "string" },
            { key: "city", type: "string" },
            { key: "state", type: "string" },
            { key: "zip", type: "string" }
          ]}
        ]            
      }
    });
  </script>
</polymer-element>
```

This would give the location input the name ```user[location_attributes][city]```. If you were 
to set ```multiple: true``` the name would become ```user[location_attributes][0][city]```. And if 
you were to set ```allowAdd: true``` the inputs would be in a list with the option to create more. 


##GROUPS AND STEPS

Sometimes you want to style a group of elements in a form and polymer selectors can get somewhat 
unwieldy without having a list of them so polymer-rails-forms now includes groups. Groups are similar to 
nests but don't change the scope for inputs, they're purely for styling. If, for instance, ```location```
in the previous example were not a nested attribute, but just a group of fields you wanted to add some special
styling to you could write it like

```javascript
  this.structure = [
    { key: "email", type: 'string', label: "Email Address", required: true },
    { key: "password", type: 'password', required: true },
    { key: "location", type: 'group', structure: [
      { key: "address", type: "string" },
      { key: "city", type: "string" },
      { key: "state", type: "string" },
      { key: "zip", type: "string" }
    ]}
  ]
```

which would keep all the address fields in the default scope but wrap them in a div with the class 'location'.

Additionally, if you wanted to make the form a multi-step form, with the user entering in the email, password 
on the first step and the location information in the second simply group fields by step like so:

```javascript
  this.structure = [
    { key: "header": type: "html", content: "<h4>This will stay visible because it's outside of the steps</h4>" },
    { key: "step_1", type: "step", structure: [
      { key: "email", type: 'string', label: "Email Address", required: true },
      { key: "password", type: 'password', required: true },
      { key: "location", type: 'group', structure: [
    ]},
    { key: "step_2", type: "step", structure: [
      { key: "address", type: "string" },
      { key: "city", type: "string" },
      { key: "state", type: "string" },
      { key: "zip", type: "string" }
    ]}
  ]
```

##Validations

Also not that I've included ```required: true``` on the email and password fields. This means this triggers
them to be validated onSubmit and onChange. For a custome validation just use ```validates: 'method_name'```

####Note:

If you're going to use the domReady or created function in your custom form element, be sure to call
```this.super()``` at the top of the function to run the form's default domReady functionality 
(which includes appending all the inputs so... kind of important). If you don't want the default 
functionality though, just make sure you run the ```this.appendInputs()``` so the inputs get added. 

##XHR

To make the form ajaxy just include the form element's ```xhr='true'``` param 
and override the ```handleXhrCallback``` function. 

##Notes about certain field types

###Arbitrary HTML

Sometimes you want to just put a bit of explanation or whatever else in the middle of a form. To
do this just create a field with the type 'html' and give is a 'content' attribute. The content attribute is 
just a string of HTML that will be injected inbetween whatever inputs you put it between. 

###Selects

Selects are the same as any other input except that they also require the values property which is and array of 
arrays, so ```[[value1, text2], [value2, text2]]```. If ```multi``` is set to true then it'll be a multi select 
with those slick paper-checkboxes

###JSON fields
Since Postgres now has those awesome HStore and JSON fields you can take for advantage of them by using a json field 

```javascript
{ key: "awesome_json", type: "json", multiple: false, structure: [
  { key: "first_field", type: "string"},
  { key: "nests_work_to", type: "nest", multiple: true, allowAdd: false, structure: [
    { key: "cool_right", type: "string" }
  ]}
]} 
```
This will create multiple inputs that all automatically serialize to a single field in the form's data called 
'awesome_json' whenever any of them are changed. If anyone out there wants to try this out with some other noSQL 
DB's that'd be awesome.


##What's supported what's not

###So far the field types that are supported are:

* string
* password
* hidden
* textarea
* email
* url
* integer
* select (single / multi)
* date (uses pickaday.js)
* location (uses google places API, which you'll have to include separately)
* file 
* image 
* checkbox
* json 
* arbitrary HTML

###What's not supported 

* radio buttons
* ranges
* everything else

Support for radio buttons, ranges (sliders) will be coming when someone requests them or I
need them for something.  

