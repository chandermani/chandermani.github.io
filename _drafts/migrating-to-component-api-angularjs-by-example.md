While Angular 4 is the talk of the town, there is still a sizeable developer community still building and supporting apps on AngularJS. If you are inheriting such a legacy (AngularJS) codebase and are new to AngularJS, you would be inheriting some good and some bad parts of AngularJS. The good being AngularJS 1.5+ component API and the bad being $scope soup. 

If you are developing something new on AngularJS 1.5+ today it becomes imperative that you embrace the component paradigm and now fallback on controller/view pair with $scope flying all around. 

But if you are to maintain and fix codebase that is making extensive use of scopes you do need to understand the nuances of scope and prototypal inheritance. This is where books released pre AngularJS 1.5+ can be helpful. The book I wrote AngularJS By Example follows the same traditional approach of using controller/view pair and scopes to pass data around. While I advice against following the approach outlined in the book, it is important you read it if you are working with legacy code.

While i wanted to update the content of AngularJS By Example and rewrite the sample application using the new component paradigm, I never spend enough time on it. With this post i plan to kickoff this migration process. 

The book has two apps on AngularJS. 7 Minute Workout (aka Workout Runner) and its extension Workout Builder. Through the series of blog post I plan to migrate the app to use the new component API.

The only reason you should read this series is, if you are working on AngularJS codebase that you plan to migrate to the new component API approach.

Let's get started!


