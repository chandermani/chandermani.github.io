There is no denying the fact how invaluable code reviews are. To list a few, code reviews helps in:
- Improving the quality of the code
- Identify exiting bugs and reduce potential bugs
- Identify gaps in requirements
- Identify gaps in design
- Junior team members learn from the seasoned peers
- Evaluate dev skills
- Learn from others
- Teach others

And maybe some more.

This post is not about how to do code review. There a wealth of information out there already. 

What I want to share are my thought around what I believe are things that help in making the complete code reviews more effective.

## Code review without the code author
The best way to do code reviews would be to sit with the developer who has written the piece of functionality and let him walk you through the code.

But if I can review someone's code independently without any help from the author I consider that piece of code well organized. The code might still have flaws but, more often than not, there is *separation of concern* (unless it's a trivial piece of code). Hence for me, this becomes a pre-condition for a code review. 

A number of times in past I have stopped code reviews in middle if I have to constantly take inputs from the developer to understand what the code is doing. When this happens its time to help our team member to re-organize his/her code. This can also happen if there is a design flaw and hence that too needs to be addressed.

The ability to review a piece of code independently becomes critical when we start to work in geographically distributed teams working across timezone. Getting two people together when they are not co-located is a challenge. 

## Code can be review without running it
## Run code analysis before any review.
## Best way to review and refactor code is fix bug related to the functionality you want to review.