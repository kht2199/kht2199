---
description: 'http://learn.knockoutjs.com/'
---

# Knockout.js

## Knockout JS?

 웹 개발 환경이 달라지면서 DOM을 직접 컨트롤하는 JQuery를 사용하고 싶지는 않고, AngularJS나 ReactJS를 쓰기에는 부담스러워 간단한 View-Model 라이브러리를 찾다가 발견한 라이브러리다. 서버단에서 컴파일 할 필요없이 가볍게 사용할 수 있는 장점이 있지만, 메이저 라이브러리가 아니기 때문에 공용 프로젝트에서는 사용할 일이 없을 것이고 개인적으로 쓰기에도 브라우저에서 No Build로 ReactJS를 사용하는게 낫지 않을까 생각된다.

## Html, View Model 정의

### HTML

```text
<!-- This is a *view* - HTML markup that defines the appearance of your UI -->
<p>First name: <input data-bind="value: firstName" /></p>
<p>Last name: <input data-bind="value: lastName" /></p>

<p>First name: <strong data-bind="text: firstName"></strong></p>
<p>Last name: <strong data-bind="text: lastName"></strong></p>
<p>Full name: <strong data-bind="text: fullName"></strong></p>

<button data-bind="click: capitalizeLastName">Go caps</button>
```

### View Model Object, Bind VM to DOM

```text
function AppViewModel() {
    this.firstName = ko.observable("Bert");
    this.lastName = ko.observable("Bertington");
    this.fullName = ko.computed(() => this.firstName() + " " + this.lastName(), this);
    this.capitalizeLastName = () => this.lastName(this.lastName().toUpperCase());
}

// Activates knockout.js
let model = new AppViewModel();
ko.applyBindings(model);
```

## Working with Lists and Collections

input 필드 값 변경시, 즉시 변경하도록 하기 위해서는 valueUpdate에 afterkeydown을 지정한다.

```text
<h2>Your seat reservations (<span data-bind="text: seats().length"></span>)</h2>
<h3 data-bind="visible: totalSurcharge() > 0">
    Total surcharge: $<span data-bind="text: totalSurcharge().toFixed(2)"></span>
</h3>
<table>
    <thead><tr>
        <th>Passenger name</th><th>Meal</th><th>Surcharge</th><th></th>
    </tr></thead>
    <!-- Todo: Generate table body -->
    <tbody data-bind="foreach: seats">
        <tr>
            <td data-bind="text: name"></td>
            <td data-bind="text: meal().mealName"></td>
            <td data-bind="text: meal().price"></td>
        </tr>
       
    </tbody>
</table>
<table>
    <thead>
        <tr>
            <th>Passenger name</th><th>Meal</th><th>Surcharge</th><th></th>
        </tr>
    </thead>
    <tbody data-bind="foreach: seats">
        <tr>
            <td><input data-bind="value: name, valueUpdate: 'afterkeydown'" /></td>
            <td>
                <select data-bind="options: $root.availableMeals, value: meal, optionsText: 'mealName'"></select>
            </td>
            <td data-bind="text: formattedPrice"></td>
        </tr>
    </tbody>
</table>
<button data-bind="click: addSeat, enable: seats().length < 3">Reserve another seat</button>

// Class to represent a row in the seat reservations grid
function SeatReservation(name, meal) {
    var self = this;
    self.name = ko.observable(name);
    self.meal = ko.observable(meal);
    self.formattedPrice = ko.computed(function() {
        var price = self.meal().price;
        return price ? "$" + price.toFixed(2) : "None";
    });
}

// Overall viewmodel for this screen, along with initial state
function ReservationsViewModel() {
    var self = this;

    // Non-editable catalog data - would come from the server
    self.availableMeals = [
        { mealName: "Standard (sandwich)", price: 0 },
        { mealName: "Premium (lobster)", price: 34.95 },
        { mealName: "Ultimate (whole zebra)", price: 290 }
    ];

    // Editable data
    self.seats = ko.observableArray([
        new SeatReservation("Steve", self.availableMeals[0]),
        new SeatReservation("Bert", self.availableMeals[1])
    ]);
    
    self.addSeat = function() {
        self.seats.push(new SeatReservation("", self.availableMeals[1]));
    }
    
    self.removeSeat = function(seat) { self.seats.remove(seat) }
    
    self.totalSurcharge = ko.computed(function() {
       var total = 0;
       for (var i = 0; i < self.seats().length; i++)
           total += self.seats()[i].meal().price;
       return total;
    });
}
let vm = new ReservationsViewModel();
ko.applyBindings(vm);
```

## Custom Bindings

```text
<h3 data-bind="text: question"></h3>
<p>Please distribute <b data-bind="text: pointsBudget"></b> points between the following options.</p>

<table>
    <thead><tr><th>Option</th><th>Importance</th></tr></thead>
    <tbody data-bind="foreach: answers">
        <tr>
            <td data-bind="text: answerText"></td>
            <td><select data-bind="options: [1,2,3,4,5], value: points"></select></td>
            <td data-bind="starRating: points"></td>
        </tr>    
    </tbody>
</table>

<h3 data-bind="fadeVisible: pointsUsed() > pointsBudget">You've used too many points! Please remove some.</h3>
<p>You've got <b data-bind="text: pointsBudget - pointsUsed()"></b> points left to use.</p>
<!-- <button data-bind="enable: pointsUsed() <= pointsBudget, click: save">Finished</button> -->
<!-- jqbutton bind. -->
<button data-bind="jqButton: {enable: pointsUsed() <= pointsBudget}, click: save">Finished</button>

ko.bindingHandlers.fadeVisible = {
    init: function(element, valueAccessor) {
        // Start visible/invisible according to initial value
        var shouldDisplay = valueAccessor();
        $(element).toggle(shouldDisplay); // to ensure the element's initial state matches the initial viewmodel data
    },
    update: function(element, valueAccessor) {
        // On update, fade in/out
        var shouldDisplay = valueAccessor();
        shouldDisplay ? $(element).fadeIn() : $(element).fadeOut();
    } 
};

ko.bindingHandlers.jqButton = {
    init: function(element) {
       $(element).button(); // Turns the element into a jQuery UI button
    },
    update: function(element, valueAccessor) {
        var currentValue = valueAccessor();
        // Here we just update the "disabled" state, but you could update other properties too
        $(element).button("option", "disabled", currentValue.enable === false);
    }
};

ko.bindingHandlers.starRating = {
    init: function(element, valueAccessor) {
        $(element).addClass("starRating");
        for (var i = 0; i < 5; i++)
           $("<span>").appendTo(element);
           
        // Handle mouse events on the stars
        $("span", element).each(function(index) {
            $(this).hover(
                function() { $(this).prevAll().add(this).addClass("hoverChosen") }, 
                function() { $(this).prevAll().add(this).removeClass("hoverChosen") }                
            ).click(function() { 
               var observable = valueAccessor();  // Get the associated observable
               observable(index+1);               // Write the new rating to it
            });
        });
    },
    update: function(element, valueAccessor) {
        // Give the first x stars the "chosen" class, where x <= rating
        var observable = valueAccessor();
        $("span", element).each(function(index) {
            $(this).toggleClass("chosen", index < observable());
        });
    }
};

function Answer(text) { this.answerText = text; this.points = ko.observable(1); }

function SurveyViewModel(question, pointsBudget, answers) {
    this.question = question;
    this.pointsBudget = pointsBudget;
    this.answers = $.map(answers, function(text) { return new Answer(text) });
    this.save = function() { alert('To do') };
                       
    this.pointsUsed = ko.computed(function() {
        var total = 0;
        for (var i = 0; i < this.answers.length; i++)
            total += this.answers[i].points();
        return total;        
    }, this);
}

ko.applyBindings(new SurveyViewModel("Which factors affect your technology choices?", 10, [
   "Functionality, compatibility, pricing - all that boring stuff",
   "How often it is mentioned on Hacker News",    
   "Number of gradients/dropshadows on project homepage",        
   "Totally believable testimonials on project homepage"
]));
```



