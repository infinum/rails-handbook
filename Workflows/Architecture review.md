# The reasons behind it

Before you start programming, your application should go through an architecture review.
When we say architecture review, we mostly have the database design and API in mind.

We do architecture planning and review in order to:

* have a clear overview of the project before we start coding
* have a clear way of communicating the project architecture to the client
* prevent scope creep
* have better architecture

**Database design is by far the most important part of the web application development process.**

# Architecture review timeline

Please follow this timeline:

1. A project manager asks you to go into an initial project meeting.
2. You should never go alone as the only Rails developer in the meeting—always take another team member with you.
3. There are no stupid questions in these meetings. It's better to understand everything at this point, than make assumptions two weeks later.
4. Ask the project manager to give you the project specification.
5. Once you're done with the initial project meeting and have the specification, sit down with the member of your team that was at the meeting with you and design the database. You can do this on a whiteboard or a piece of paper, or use a tool, such as [Gliffy](https://www.gliffy.com/) or [MySQL Workbench](https://dev.mysql.com/downloads/workbench/). If you need to draw a sequence diagram, use [Web Sequence Diagram](https://www.websequencediagrams.com/).
6. Consult the project manager at will and add new discoveries to the specification if you find that something's missing.
7. Don't hesitate to involve other team members since this is the most important part of the app.
8. Once you've finished the database design, it has to be approved by someone from the management.
9. Once your architecture has been approved, you can generate a new Rails project. Read more about that on the next page.

Ask the project manager to give you the project specification.

## Database design & discuss major implementation details
Once you're done with the initial project meeting and have the specification, you can start designing the database. Use an online tool so you can easily present the diagram later on the review, such as [dbdiagram](https://dbdiagram.io/home) or [SqlDBM](https://sqldbm.com/Home/), or some other.

Consult the project manager at anytime and add new discoveries to the specification if you find that something's missing. Don't hesitate to involve other team members since this is the most important part of the app.

When you're done with the database diagram, sit down with the member of your team that was at the meeting with you and go through:
* the database diagram
* authentication
 - what gems will be used
 - API authentication - cookies (web frontend) or jwt/custom tokens (mobile frontend), or both
* admin UI setup
* emails - templates (preferred) or custom design (rarely a good use of time for MVP)
* any 3rd party services
* any other project specific problems

## The review meeting
Once you've finished the database design and discussed other major implementation details with the team member, it has to be approved by the project sponsor. The next step is to ping the project sponsor to organise the review meeting. At the meeting the main focus will be on the database design. Standard implementation details will be covered briefly. This meeting is a good place to dicuss any other uncertainties or problems.

## Start coding
Once your architecture has been approved, you can generate a new Rails project.
In order to start coding, the whole project must go through an architecture review.

Sometimes, you will notice some obvious mistakes you couldn't have seen when you started the project. If these make a large architecture overturn, notify the project sponsor to agree on the next steps.

**Think of this as a never bending rule.**
