---
layout: post
title:  "O'Reilly Conference Notes"
date:   2020-11-17 20:48:20 -0700
categories: notes
---
This past week I watched a good chunk of the 2019 and 2020 O'Reilly software architecture conference. This post is mostly a record of what I learned, but if anyone else finds it interesting and learns something from it, that's great, too!

# Career advice for architects - Trisha Gee (JetBrains)

The main point I got out of Trisha's talk is that the so-called "soft skills" are absolutely crucial for success as a software architect. Architects often don't have direct reports, and even when they do, their influence matters far more than their authority.

Trisha also discussed how architects can either try to scale their skills vertically or horizontally. Vertical scaling would be trying to update your own skills to the point where you can do everything. Horizontal scaling involves teaching others to be able to do everything you can do.

The most effective way to scale horizontally, in her opinion, is to pair program. Mob programming can be used as well.

Other things she suggested:

* Internal learning sessions--sharing things you've learned with colleagues.
* Book clubs--reading books together as a team. To avoid everyone having to read the whole book, everyone could read a single chapter and present on it for 5 minutes. Feedback sessions are absolutely necessary.

# Design and architecture: Special Dumpster Fire Unit - Matt Stine (Pivotal)

Matt Stine went over a few failed architectures and briefly analyzed what happened with each of them. A few key takeaways:

* Don't go distributed if you don't have to!
* Learn from history! Don't repeat mistakes of the past. Document your past decisions, successes, and failures.
* Think in the long term. Avoid project thinking, use product thinking.
* Avoid the new shiny things unless they *really* solve a problem you have.
* Validate your assumptions!

# Roaming free: The power of reading beyond your field - Glenn Vanderburg (First.io)

Glenn's talk focused on reading books outside your field. Learn about how other fields work, as that can teach you things applicable to your field.

He quotes Steve Jobs:

> Creativity is just connecting things. When you ask creative people how they did something, they feel a little guilty because they didn't really do it, they just saw something. It seemed obvious to them after a while.
>
> That's because they were able to connect experiences they've had and synthesize new things. And the reason they were able to do that was that they've had more experiences or they have thought more about their experiences than other people.
>
> Unfortunately, that's too rare a commodity. A lot of people in our industry haven't had very diverse experiences. So they don't have enough dots to connect, and they end up with very linear solutions without a broad perspective on the problem. The broader one's understanding of the human experience, the better design we will have.

We can't experience everything, but reading books from outside our field can help to seed ideas from other industries.

Some books that I want to read after watching this talk:

#### *The Design of Everyday Things*, Don Norman

This one has been recommended to me a bunch, and I think it's high time I read it.

#### *How Buildings Learn: What happens after they're built*, Steward Brand

This book seems particularly relevant as we try to cope with architectures that change over time. The author describes how buildings change over time, and what that means for architects.

#### *The Mangle of Practice*, Andrew Pickering

This book discusses how messy processes can actually lead to good outcomes. A very interesting topic given the state of software practices today!

#### *Cognition in the Wild*, Edwin Hutchins

This book describes how teams of people learn as a group. Teams learn differently than individuals, and we develop mechanisms for preserving that knowledge.

#### *Emergence*, Steven Johnson

Related to the above, the concept of emergence has been fascinating to me lately, and this seems like a good option for learning more.

# Technical Debt: A Master Class - Robert "r0ml" Lefkowitz (Retired)

R0ML argues that we normally call technical tech isn't really debt, as there's no sense of borrowing. When do we borrow in software? When we use dependencies! Therefore, real technical debt comes from adopting dependencies, as we borrow someone else's code that we now have to maintain (whether by updating it ourselves or constantly upgrading versions), or else risk a failure in their code breaking our code.

He makes a very compelling argument against adopting dependencies unless they are sufficiently complicated that it's really infeasible to do it yourself. His threshold for that is 5 million lines of code, but that seems a bit steep to me, to be honest. I think there are problems that are simpler than that that you still don't want to deal with.

An interesting approach to removing dependencies that he suggests: delete the library, then see what gives errors. Copy just those functions into your own codebase (with appropriate license information). You now own that code, it's your responsibility to understand it and maintain it. The advantage here is that you only are responsible for the bits that you actually use, not the entire framework.

In the second half of his talk, he proposes R0ML's Laws of Software Development:

* First Law: The number of lines of code in your system(s) divided by 25,000 equals the number of people in your technology organization.
* Second Law: A software developer will generate 1,000 lines of debugged code per month.

He then combines these into R0ML's First Theorem of Software Development:

* A software developer has a useful life of two years. After generating approximately 25,000 lines of code, the rest of their tenure is spent maintaining those 25,000 lines.

This is amusing, but actually seems pretty accurate from my limited experience, and he backs it up with numbers from a bunch of organizations. The application of this: reduce the lines of code in your code base!

In the rest of the talk, he emphasizes that real refactoring should consist primarily of improving your types. A few quotes he uses:

> I will, in fact, claim that the difference between a bad programmer and a good one is whether he considers his code or his data structures more important. Bad programmers worry about the code. Good programmers worry about data structures and their relationships.
>
> -- Linus Torvalds

> Show me your flowcharts and conceal your tables, and I shall continue to be mystified. Show me your tables, and I won't usually need your flowcharts; they'll be obvious.
>
> -- Fred Brooks

#### Safe microservices

A really interesting idea that he draws from this refactoring-as-data-change principle is that if you look at it through this lens, you can see a way to get many of the benefits of microservices without actually going distributed. If you separate your data into separate databases, and keep your bounded contexts in their own databases, then you get the separation and modularity of microservices without any of the distribution.

Another book recommendation: *Understanding Computers and Cognition*, Terry Windograd and Fernando Flores.

Another point: reusable components take longer to write and longer to maintain. A good rule of thumb: if you need to modify more than 20% of the existing code to add a new parameter or whatever, then you're better off writing a new one.

# An architect's guiding principles for leadership - Seth Dobbs (Bounteous)

I'm actually combining the notes on this talk with the notes from his Tutorial session later on, which basically elaborates on the same ideas.

Seth Dobbs emphasizes principles--he actually sounds a lot like he's drawing from Steven Covey in a lot of ways.

His definition of leadership was really interesting to me:

> The goal of leadership is to *influence* individuals, teams, and organizations to *effectively* deliver *durable* results.

#### Shape the problem first

In pursuing this, Seth argues that the biggest thing leaders can do is to help to shape the problem before the solution. "A problem well-stated is a problem half-solved." If you start trying to solve the problem before you've really defined it, you will most often end up solving the wrong problem.

> Our solutions are valuable only if our business / clients / users see them as solving meaningful problems. As architects, we need to tie technical needs to business problems.

This doesn't involve lying to the business. If there is no business problem being solved, then the problem probably isn't worth solving. It's a question of showing the business how a tech problem affects the business. Therefore, a good problem statement identifies desired outcome that is missing (or undesired outcome that is present) and addresses the actual business value at stake. It also considers the context.

#### Create and share vision

Having defined the problem, a leader's task is to help share the vision. Seth emphasizes that by "vision" he doesn't mean the kind of grandiose vision statements that often pervade corporate-speak. What he means is simply helping the team to see why we're doing what we're doing. The business has needs, and we try to address those needs. Alternatively, we're pursuing a certain technical goal.

Seth argues that for a vision statement to be effective, it has to address all stakeholders: business, users (and customer support), technical. Each needs to see how it fits into their overall goal. Vision is outcome-focused.

To find the vision, we may need to ask: the business may not know "why". For example, they may say "we need to optimize SEO", but what do they actually hope to accomplish by that? This goes along with shaping the problem.

> If we don't have a common measuring stick for checking the value of our work, all things are equal and decision making becomes arbitrary.

Steps for creating and communicating vision:

1. Research: Understand the "Why". Identify who the stakeholders are, and what they are looking for.
2. Qualify: Establish a clear context for your vision. This includes shaping the problem statement, surfacing and addressing your assumptions, and identifying constraints.
3. Define: Identify the "what" and "why", ignoring the "how".
4. Communicate: Share the vision with others, and **get feedback**.

> The hardest lesson for many managers to face is that ultimately there is really nothing you can do to get another person to enroll or commit. They require freedom of choice.
>
> -- Peter Senge

An example vision statement from my own work:

> We will implement Event Sourcing to bring visibility into the flow of data through the system. This will assist in debugging, because we will be able to see user actions over time. It will provide greater insight into retention and other processes, allowing more informed decisions. It will allow customers to audit changes to their data over time, and allow customer support to better help them fix problems.

#### Problem solving

Another skill he discusses is problem solving. His proposed framework is basically the scientific method: develop your problem statement, then create a vision to describe a solution to the problem. At that point, attack that vision with everything you've got. Ask disproving questions, find any hidden assumptions. Identify risks and tradeoffs. Is this really the solution?


#### Tension versus Friction

Seth distinguishes between tension (conflict of ideas) and friction (conflict of people). It's normal and healthy for people to disagree, as long as it stems from positive intent and everyone is working together toward a goal.

When you have a vision that survives the gauntlet, then move forward with it and solve your problem.

#### Make yourself obsolete

Another interesting idea from Seth: make yourself obsolete. If you can train someone else to do your job, you're doing more for the organization than if you try to keep doing it yourself.

# The well-rounded architect - Patrick Kua (N26)

This is similar to other talks on balancing soft and hard skills. Patrick describes the well-rounded architect as balanced in the following categories:

* Communicator: effectively communicating with business needs.
* Leader: able to influence technical team.
* Developer: a good developer in their own right.
* Systems Focused: able to see how all the pieces fit together.
* Entrepreneur: a solid understanding of costs and benefits.
* Strategic Technologist: not chasing new shinies, but keeps a tech radar active.

# Beyond accidental architecture - James Thompson (Cingo Solutions)

My key takeaways from James's talk had to do with documentating architectural decisions. He recommended looking at [the C4 model](https://c4model.com/) and [ADR](https://adr.github.io/) for specific structures of documentation.

ADR seemed especially interesting to me. The basic idea is to track every architectural decision in the repo, so you can always go back and see what you decided the last time a given subject came up.

He recommended that documentation have a few key features:

* All in one place (near the code!)
* Easy to update
* Perishable--give it an expiration date.

On the last point, the idea is to just put a "Best By" date on the documentation, so future developers know if something likely needs to be updated (whether because of framework updates or simple code changes).

# Beyond the technical: Small steps to playing bigger (aligning teams focus with stakeholders targets) - Maggie Carroll (MAG Aerospace)

Maggie's talk was about building influence. Software architects can't just hand down architecture.

She divides "building influence muscles" into three steps: Lead, Influence, Act. I've recategorized these items in a way that makes sense to me: Lead is the core principles, Influence is things we do with others, Act is things we do ourselves.

#### Lead
* Control your own work framework.
* Authority must be earned by influence.
* Keep focused on business value.
* Create a "business rhythm". Predictable, structured approach to the week.
* Evaluate and plan weekly for 1 hour (Friday), going through each "Act" item.

#### Influence
* Ask questions that help to clarify and enhance a mindset.
* Rely on other experts. Don't just give them the solution.
* Ask questions that help to clarify a mindset.
* Review the artifacts with stakeholders. Do they still line up with goals?
* Review meeting goals and frequency with stakeholders.
* Communicate with stakeholders on what you find in weekly analysis.

#### Act
* Identify shared goals.
* Identify stakeholders--list them all out. All customers and internal stakeholders.
* Identify stakeholders' alignments to the shared goals.
* Identify Key Artifacts. These are pieces of paper (or digital artifacts) that are used to shape the project. A roadmap, a list of key principles are good examples.
* Identify goals for meetings

# So you think you might be an architect - Sonya Natanzon (Guardant Health)

Sonya breaks down an architects role into 3 categories: People, Technical, and Thought:

* People
  * Influence
  * Share knowledge
  * Grow talent
* Technical
  * Learn
  * Build a scaffold
  * Select a toolbox
* Thought
  * Evolve the big picture
  * Champion process
  * Set culture and values
