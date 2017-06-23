While Angular 4 is the talk of the town, there are developer still building and supporting apps on AngularJS. If you are new to AngularJS and have started working on an existing AngularJS codebase, you would be inheriting some good and some bad parts of AngularJS. The good being component API (AngularJS 1.5+), data-binding framework, DI, services and few other. The bad being the scope soup. 

If you are starting something new on AngularJS 1.5+ today embrace the component paradigm and stay away from the approach of using controller/view pair with scope flying all around. 

But if you have to maintain/fix codebase from the pre component era, then books released pre AngularJS 1.5+ can be really helpful. The book I wrote AngularJS By Example followed the same traditional approach of using controller/view pair and scopes to pass data around. 

An update to the content of AngularJS By Example is long due. If I had to write the sample applications from the book again I would had used the component paradigm. While the book rewrite is not something i am planning, I do want to migrate the companion apps to use AngularJS components. 

This blog post kicks of my migration journey. This series should help everyone who want to migrate their AngularJS app to use the component approach.

The book has two apps on AngularJS. 7 Minute Workout (aka Workout Runner) and its extension Workout Builder. We will migrate each one of these.

Let's get started!

# The Process
The book as total of 9 chapters where we gradually build the app from the grounds up. 
The code is available on my GIT repo
https://github.com/chandermani/angularjsbyexample

As the book incrementally builds on the sample apps, and there are number of checkpoints (implemented as git branches) 
that bookmark the state of the code. This allows reader to match his implementation to what the book provides. 

I will take the final checkpoint on each chapter and migrating the code at that point in time to the new component approach.

# Migrating Chapter 2, Building our first app - 7 Minute workout
7 Minute Workout is an exercise/workout plan that requires us to perform a set of twelve exercises in quick succession within the seven minute time span. At the end of Chapter 2, this is how the app looks.

I suggest you clone the codebase form the github repo, and switch to branch checkpoint2.7. 
git clone https://github.com/chandermani/angularjsbyexample
git checkout checkpoint2.7

This is where we start our migration. Create a new branch (from checkpoint2.7) checkpoint2.7-migrate-ng-component.
git branch checkpoint2.7-migrate-ng-component

Navigate to the folder trainer and have a look at what has been implemented thus far. The easiest way to run the app would be to use local http server to serve the start page (`index.html`). 

```bash
npm install -g http-server
cd trainer/app
http-server
```

Even if you have not read the book, the implementation is quite simple. The view reside in `partial` folder and the app implemenation in `js` folder. WorkoutController (in js/7MinWorkut/workout.js) does most of the heavy lifting. Spend some time playing with the app and looking at the code in app.js and workout.js.

## Upgrading to the latest Angular version
Since world have moved onto node/npm ecosystem, we better off get our dependencies from npm instead of relying on CDN.

Add a new package.json file from [here](https://github.com/chandermani/angularjsbyexample/tree/checkpoint2.7-migrate-ng-component/trainer/app) to you local trainer/app folder, and run `npm install`.

Update index.html to point to the new Angular and Bootstrap dependencies. You can copy it from here again (https://github.com/chandermani/angularjsbyexample/tree/checkpoint2.7-migrate-ng-component/trainer/app)

With the framework upgrade we also need to fix the route url in start.html and finish.html page to use the hashbang mode (#!) instead of hash (#) to define url.For example `<a href="#/workout">` becomes `<a href="#!/workout">`.

Refresh the application to make sure the upgrade went fine.

## Migrating WorkoutController
We are going to take a top down approach to migration. The top level view/controller pair will be migrated first (#!/workout). Once we create a component from workout view/controller we can see if there are possibilites to create child components inside the workout component.

Rename the file workout.js to workout.component.js and update index.html reference for the script file.

This movement towards building component also affects the app's folder structure. Instead of using a common folder `partials` to store all views, we store everything related to a component within the component folder, which includes the underlying views, sytle sheets, and any child views.

So go ahead and move `partials/workout.html` to `js/7MinWorkout/workout.html`

Lets focus our attention now on `WorkoutController`. A quick refresher on the general idea would help.

### AngularJS Components
In AngularJS components widgets that are defined as html tags, render their own content and can take inputs. Traditionally this is how we define our controller and view:

```javascript
function UserProfileController($scope) {
    $scope.user.name = "John";
}
```
```html
<div id='user profile' ng-controller='UserProfileController'>
    Welcome <span>{{user.name}}</span>
</div>
```
The linking happens through  `ng-controller` directive applied on the view html.
But with the component approach the component definition encapsulates the complete details about the component. 
```javascript
function UserProfileController() {
    this.user.name = "John";
}
 
angular.module('main')
    .component('userProfileComponent', {
        template: "<div id='user profile' ng-controller='UserProfileController'> Welcome <span>{{username}}</span></div>",
        controller: UserProfileController,
        userDetails:"<"
    });
```
```html
<user-profile-component user-details="user"></user-profile-component>
```

Just by looking at the component definition we can make out the component's view , the input it takes (`userDetails`) and the controller implemenation. Clearly a component is a self contained reusable widget.

Once we adopt the new component pardigm our application is nothing but a hierarchy of components, sometimes called the component tree. 

Time to create a new workout component. Open `workout.component.js` and add the component definition:

```javascript
angular.module('7minWorkout')
    .component('workoutRunner', {
        templateUrl: 'js/7minworkout/workout.html',
        controller: WorkoutRunnerController
    });
```

We don't have `WorkoutRunnerController` that implements the component logic as yet. But we do have a AngularJS controller registration (`.controller('WorkoutController'...`). Create a new constructor function and move all the code from the AngularJS controller registration into the new function:

```javascript
function WorkoutRunnerController() {
    <existing controller>
}
```
Now is the time to refactor the new contoller implemation to support the component paradigm. This requires
- Getting rid on scope ($scope) usage.
- Injecting the other dependencies into the new controller.

Easy stuff first, injecting everything other than $scope into WorkoutRunnerController

WorkoutRunnerController.$inject = ['$interval', '$location'];
function WorkoutRunnerController($interval, $location) {

We use the static $inject property here to define injection.

Since the $scope injection is gone, the controller needs to be fixed.

### Getting rid of scope
In the component world the controller defines the state that the component view binds to. The controller itself is the viewmodel. Therefore we need to get rid of the scope by attaching the properties + functions that were previously defined on scope to the controller instance.

Inside the construction function take hold of the controller instance (into `ctrl`):

```javascript
function WorkoutRunnerController($interval, $location) {
    var ctrl = this;
```
And do a find and replace for `$scope` -> `ctrl` and we have got rid of scope! We could have used `this` instead of `ctrl`, but everyone knows `this` reference changes based on context/invocation (such as `$interval.then` callback function where `this` points to the window).

### Using component lifecycle hook $onInit for initialization
Components have well defined lifecycle hooks, which are called at certain points in the life of the component. We can use the $onInit hook to replaces our custom initialization code. This means, removing this:

```javascript
var init = function () {
          startWorkout();
      };
```
And moving the initialization code to $onInit

```javascript
ctrl.$onInit = function () {
        startWorkout();
    };
```

The component backend is complete, but the view bindings need to change as properties are now defined on the controller, not scope.

## Fixing view bindings
There is one think that I m Angular UI bindings depend upon the scope. Angular exposes the component controller on the view through the convinient property `$ctrl`. Internal what Angular is do







