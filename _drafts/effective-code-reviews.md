There is no denying the fact how invaluable code reviews are. To list a few, code reviews helps in:
- Improving the quality of the code
- Identify exiting bugs and reduce potential bugs
- Identify gaps in requirements
- Identify gaps in design
- Help junior team members learn from seasoned peers
- Evaluate dev skills
- Learn from others
- Teach others

And maybe some more.

This post is not about how to do code review as there a wealth of information out there already. Instead what I want to do here is to share my thought around what I believe are prerequisites to an effective code review.

## Code review without the code author
The best way to do code reviews would be to sit with the developer who has written the piece of functionality and let him walk you through the code.

But if I can review someone's code independently without any help from the author I consider that piece of code well organized. The code might still have flaws but, more often than not, there is *separation of concern* (unless it's a trivial piece of code) in place. Hence for me, this has become the first litmus test for any code review. 

Quite often in past I have stopped code reviews in middle if I had to constantly take inputs from the developer to understand what the code is doing. When this happens its time to help the team member to re-organize his/her code. This can also happen if there is a design flaw and hence that too needs to be addressed.

The ability to review code independently also becomes critical when working in geographically distributed teams spread across timezones. Getting two people together when they are not co-located is always a challenge. 

## Code review without execution
A complete no brainer! If a piece of code requires me to execute it before I can make any sense of what's happening then the code is flawed. If this happens it's again time to get the author/developer involved and refactor/reorganizes the code. 

The is no well defined recipe on how to fix such an implementation but the eventual goal is to make the code and the flow predictable.

## Run code analysis before any review.
Every modern language/framework out there has some code analysis tool(s) that can automate parts of this process. These tools help us avoid spending time on stuff that the tool can easily identify. 

Bottom line, add code analysis step to your code check-in and CI process.

## Best way to review, fix bug related to the functionality you want to review
While this does not qualify as a prerequisite, in my opinion there is no better way to review a piece of code than to fix a bug or extend it yourself.

When we either fix a bug or try to extend a piece of code, we gain a better understanding of how the piece of code works and how (well) it connects to the other parts of the system. The better the code the easier it is to fix/extend. And if the code is tough to work with we know we have a problem at hand, **technical debt**. We do not want too much of it!