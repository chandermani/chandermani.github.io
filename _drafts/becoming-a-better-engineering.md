## Becoming a better engineer

*Good vs Bad Software Engineer*, while a cliché, was something that I wanted to write about. Thankfully I realized very early, there are no good or bad engineers, instead, there are engineers that improve over time and the ones that don't. Everyone starts the journey at various points in the badness meter, and only with persistent efforts become better at their skill. 

> A bad engineer is one who does not improve with time.

Hence I am not going to focus my energy, writing about a topic with a negative connotation. Instead what I am going to do is to put my perspective on how to improve at your craft and becoming a better software engineer. Something that I really care about.

### Know your tools

One of the metrics to measure the skills of an engineer is productivity. *Productivity* is the ability to generate high-quality results with the least amount of effort and time.

Tools play a big part in ensuring engineer productivity. A feature rich IDE, a capable debugger, a great unit testing tool/framework make our life simpler. The onus is then on us to make the best use of these tools.

Take the IDE, it's is a cardinal sin if you don't know the keyboard shortcuts for all the common task you perform every day inside the IDE.

Using a debugger (maybe part of IDE or not), then learn about its capabilities and how to debug effectively. Example, do you know the difference between *Step Into*, *Step Over*, and *Step Out*.

You can go a step further and extend your IDE with numerous useful extensions/add-ons. Many popular IDE's extensions/add-ons can make your life easier. Extensions/add-ons such as *ReSharper* (for Visual Studio), *TSLint* (for Visual Studio Code) are some good examples in this category.

The bottom line, learn to use your tools effectively and constantly look to new tools/extension that can further reduce manual effort.

### Handling Information Explosion

Even with a decade of experience under my belt, I still at times feel overwhelmed by the pace at which the technology is evolving. Keeping abreast with what's happening around can become a daunting endeavor. How do we manage this information explosion? There are a few things that have helped me keep ahead of the learning curve. These include

#### Healthy mix of new technology and fundamentals

While technology evolves every day, fundamentals remain the same (or do not evolve that rapidly). Still, there is a mad rush to learn new technologies and blatant disregard for fundamentals. These fundamentals may vary based on domain, but these are the building block for any technical expertise. Be it programming fundamentals, networking fundamentals, operating system (OS) fundamentals, web platform fundamentals we ought to have our fundamentals sorted out.

If you are a web engineer, while learning the new uber cool JS framework out there helps, you cannot ignore the building blocks of the web: HTML, CSS, JavaScript. Have you spent time understanding how CSS works? Do you understand the nuances of *this*, what is a *closure*, what is *prototypal inheritance* in JavaScript?

For general programming, there are some classics (books) out there that are almost technology agnostic. Read them! Here is a small list that I recommend (in no particular order):

- [The Pragmatic Programmer] (https://www.amazon.com/Pragmatic-Programmer-Journeyman-Master/dp/020161622X/)
- [Clean Code] (https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [Code Complete] (https://www.amazon.com/Code-Complete-Practical--Handbook-Construction/dp/0735619670)
- [Release it] (https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213)
- [Design Patterns] (https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
- [Refactoring: Improving the Design of Existing Code] (https://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672)
- [Growing Object-Oriented Software, Guided by Tests] (https://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627)
- [Test Driven Development: By Example] (https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530)

#### Follow the thought leaders

Every professional field has some thought leaders who are the best at their trade. Identify thought leaders in your area and follow them. Read what they write, explore what they endorse. Remember they have walked the path you are on now. Follow them.

#### Follow high-quality content aggregators

There are some excellent content aggregators out there. Explore them. Sites like 
- [InfoQ] (https://www.infoq.com/) - A great resource for software engineers with a focus on software development. A site that I personally use and recommend.
- [HackerNews] (https://news.ycombinator.com/)
- [TechCrunch] (https://techcrunch.com/) - for business and tech new
- [ProductHunt] (https://www.producthunt.com) - From the site *"Product Hunt surfaces the best new products, every day. It's a place for product-loving enthusiasts to share and geek out about the latest mobile apps, websites, hardware projects, and tech creations."*

Such sites do a great job of aggregating high-quality content, helping us keep ahead of the tech curve.

### Don't give up when the going gets tough

I have worked with engineers who yearn for challenges and the ones that are content with just copy/paste. The tendency of copy/paste engineers is to learn the easy parts of a technology and/or project and ignore/delay the not so easy stuff. They always looking for shortcuts/hacks when working on any nontrivial assignment. Such people focus mainly on getting things done, instead of getting things done the right way.

An example of this that is fresh in my mind is around AngularJS. We were early adopters of AngularJS. The team was new to the platform. Over time the engineers that excelled at AngularJS were the one who looked beyond the standard usage of AngularJS. They understood what prototypal inheritance worked with Angular, how AngularJS data-binding worked, how/when to create directives (HTML extensions).

To become a better engineer, you cannot be one who learning everything at a superficial level. You should develop expertise in product/technology areas that you work day in and day out. Something that I touch again later in this post.

### Understand the big picture

*Understand the big picture*. If this has not been told to you in one form or another then you have not built anything substantial! When working on a problem, more often than not, our focus is too narrow. At times we do not understand how the components we are building fits into the complete solution context. Neither are we aware of the business value of our deliverables. This leads to sub-optimal solutions and/or missed requirements. Avoid this, and always, always look at the big picture! 

Besides the solution and business value context described above, there is one more vector that I should add. Understand the big picture in term of the *software development life cycle* (SDLC).

I remember my early days, fresh out of college I had a preconception that software development is all about writing code and everything else is optional. Boy, was I wrong!

Writing code while important is just one facet of software development. There is more to it, and you need to understand the big picture here too!

To become a better engineer, learn about all aspect of building software and if possible, get involved. Learn about requirement gathering, prioritization/triaging, project/sprint planning, deployment (CI/CD), testing (automated, smoke, regression, UAT) release management, operations, support. The earlier you learn about them the better you understand the big picture. Don't restrict yourself to taking the requirement, building the software and throwing the code to the other side of the wall for someone to test. You will never grow!

### Learn about operations

Thanks to the *devops* culture, engineers are getting involved in areas which they had limited exposure earlier. This is definitely helping everyone understand the big picture and write better software.

If you have not adopted devops, then I highly encourage you to learn about the deployment and operational aspect of your software. To get better at what you are building, you need to understand where is it deployed and how is it used by the end customer. Learn about, how deployments/releases are made, the deployment topology, what operations team is doing. How are they monitoring the health of the deployment?

### Learn from mistakes

Experience is the name everyone gives to their mistakes. - Oscar Wilde
You will make mistakes! Learn from your mistake and from the mistake that you observe others making. Mistakes are part of our learning process, and as long as we learn from them and take corrective measures these mistakes give us great insights into what works, what doesn't, and why it doesn't. It is difficult to appreciate the right way of doing things unless you have made a few mistakes and suffered. But remember:
Learn from the mistakes of others. You can't live long enough to make them all yourself - Eleanor Roosevelt
A great engineer makes fewer mistakes, learns from others, and makes sure he does not repeat them again.

### Become an expert on things you are actively involved

Pick any technology domain and I bet you can spend a copious amount of time mastering it. There is so much to learn that when it comes to actually decide what's worth our time, we become indecisive. The easiest choice here is to gain in-depth expertise on areas that you are actively involved in, the technologies that your project uses, the tech stack your Org endorses. This is also a great way to stand out among your peers and align your expertise with business needs.

### Challenge the status quo

The hallmark of a skilled professional/master is that he doesn't accept the status quo. He is constantly striving for improvements in all aspects of his domain of influence. Part of this is not accepting the excuse "But we have always done it this way". This also means having an opinion and a strong foundation to support your opinion. To challenge the status quo requires us to be expert in our domain and look at everyday work challenges with a fresh perspective.

### Learn for your peers

I have saved best for the last. Pairing with a smart coworker is accelerated learning! Working with a master teaches you more than what is possible through any book/video/tutorial. Working with an expert gives you deep insight into how he approaches a problem, deconstructs it, structures and verifies his solution. If your organization is a proponent of practices such as Pair Programming great benefits can be derived by working with masters of their trade.
While this is the best form of learning it is not always possible to pair with someone at work. The next best option is to look at their work/code/solution and learn from it. The web has made this a level playing field, if there is no one around you, look at GitHub repos of smart engineers and great projects.

To conclude, I will paraphrase the opening statement:
> A good engineer is one who learns and improves over time.
And I am sure there are more ways out there that can benefit us in our pursuit for excellence.

Happy Learning!