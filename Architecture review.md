## The reason behind it

Before you start programming, your application should go through an architecture review.
When we say architecture review, we mostly have the database design and API in mind.

We do architecture planning and review in order to:

* have a clear overview of the project before we start coding
* have a clear way of communicating the project architecture to the client
* prevent scope creep
* have better architecture

**Database design is by far the most important part of a web application.**

## Architecture review timeline

Please follow this timeline:

1. A project manager asks you to go into an initial project meeting.
2. You should never go alone as the only Rails developer in the meeting—always take another team member with you.
3. There are no stupid questions in these meetings. It's better to understand everything at this point, than make assumptions two weeks later.
4. Ask the project manager to give you the project specification.
5. Once you've done the initial project meeting and got the specification, sit down with the member of your team that was at the meeting with you and do the database design. You can do this on a whiteboard, on a piece of paper, or use a tool, such as [Gliffy](https://www.gliffy.com/) or [MySQL Workbench](https://dev.mysql.com/downloads/workbench/). If you need to draw a sequence diagram, use [Web Sequence Diagram](https://www.websequencediagrams.com/).
6. Consult the project manager at will and add new discoveries to the specification if you find that something's missing.
7. Don't hesitate to involve other team members since this is the most important part of the app.
8. Once you've finished the database design, it has to be approved by someone from the management. Matej is the first person to ping.
9. Once your architecture has been approved, you can generate a new Rails project. Read more about that on the next page.

In order to start coding, the whole project must go through an architecture review.
If for any reason this is not possible, the project should be split in phases. In that case, each phase needs to go through an architecture review before coding can begin.

Sometimes, you will see obvious mistakes that you couldn't see when you started the project—if these make a large architecture overturn, go back to step 8.

**Think of this as a never bending rule.**
