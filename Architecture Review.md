## The reason behind it

Before you start programming, your application should go through an architecture review.
When we say architecture review, we're mostly thinking about the database design and the API.

We are doing architecture planning and review in order to:

* have clear overview of the project before we start coding it
* to have a clear way of communicating project architecture to the client
* to prevent scope creep
* to have better architecture

**Database design is by far the most important part of a web application.**

## Architecture Review timeline

Please follow this timeline:

1. You get prompted by a project manager to go into a initial project meeting.
2. You should never go alone as the only Rails developer in the meeting - always take another team member with you.
3. There are no stupid questions in these meetings. It's better to understand everything at this point than making assumptions two weeks later.
4. Ask the project manager to hand you out the project specification.
5. Once you've done the initial project meeting and got the specification, sit down with the member of your team that was in the meeting with you and do the database design. You can do this on a whiteboard, on a piece of paper or in a tool such as [Gliffy](https://www.gliffy.com/) or [MySQL Workbench](https://dev.mysql.com/downloads/workbench/).
6. Prompt the project manager at will and add new discoveries to the specification if you find something missing.
7. Don't hesitate to involve other team members since this is the most important part of the app.
8. Once you've done the database design, it has to be approved by someone from the management. First person to ping is Matej.
9. Once your architecture is approved, you can generate a new Rails project. Read more about that on the next page.

In order to start coding, the whole project must go trough architecture review.
If for any reasons is not possible, project should be split in phases; in that case each phase needs to go through architecture review before coding can begin.

Sometimes, you will see obvious mistakes that you couldn't see when you started the project - if these make a large architecture overturn, go back to step 8.

**Think of this as a never bending rule.**
