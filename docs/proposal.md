# Registers Refactoring Proposal

*Make things open: it makes things better*

First, I’d just like to say that I don’t wish in any way to disparage the existing work of the registers build teams — the decisions that have been made up to this point were necessary in order for us to work with real data and to learn about what real users needed. 

That said, knowing what we now know, I propose that we change how the Data Infrastructure team approaches the development of the Open Registers software platform in favour of simpler and more open technical choices. This proposal applies only to the software itself, not to the work of the Registers Design Authority, nor to the register field conventions currently defined. This proposal _does_ have implications for the way registers are stored, indexed and shared, and so to avoid migration confusion, I recommend that this refactoring be carried out before the register software reaches the Live phase.

This proposal outlines some brief findings from user research and from adoption discussions thus far, applies the [GDS Design Principles](https://www.gov.uk/design-principles) to the current registers architecture and then makes a specific proposal for refactoring.

## Proposed Changes

I propose the following changes:

* Cease development of the proprietary java application server, database, deployment environment, API and external representation of registers.
* Redefine a register to be a delimited text file (or collection of files) which can be versioned and inspected publicly. Use standard tools such as git/github to keep these files publicly available and synchronised.
* Immediately begin work on an index server layer to provide an API and common widgets that can make use of multiple, joined registers.

## Domain Model

In our limited user testing and adoption thus far, we have found out a few things from users.

* Potential and current custodians think of their registers as authoritative *lists* of values.
* Many custodians think of their data in terms of their current *spreadsheet* software — as a tabular array of rows and columns.
* More sophisticated custodians might think of their data in the form of a database — but think of their lists in terms of *tables*. Users think about small lists used to categorise things (for example, the religious character of a school) as *lookup tables*, a term drawn from the relational database world.
* Custodians recognise that their lists change over time. While there are some kinds of lists, such as a record of legal deeds, in which a list is only *appended* to over time, there are others, such as a list of schools, where the items can *change* over time. Items are also sometimes deprecated or *removed*, even from official lists.
* Custodians need to *manage* their lists of values. They might make changes because the world has changed because of new legislation, new policy or a change in a real thing, like a country name. They also might make changes for administrative reasons, such as to allow a new option or even to correct an earlier mistake in light of new information.
* Consumers of registers, such as digital service teams using a list in a web service, need to rely on the data being up to date. This seems to be a key concern, and we have had questions about subscribing to changes.
* A technical service team might represent a register as a *text file*, such as the file used to build a Countries drop-down.

Our current design differs from these expectations in a few ways. 

* We separate *Items* (as objects in a list) from *Entries*, the changes made to a list over time. We group these conceptually into *Records*.
* We model our register as primarily an API — a machine service — rather than as a file.
* We separate the storage of registers (currently in a database which is protected from public view) from the presentation of them in various transformed output formats.
* We do not enable our users to inspect the raw data of a register list; rather, we provide tools that allow them to query our register and perform certain calculations upon them to verify their consistency.
* We model the checks needed to verify entries explicitly, and make these part of the human-readable representation of the list and of the external (text) representation of the list.
* We think of a register as a data *service* that needs to be run and hosted somewhere, rather than as information.


## Design Principles Applied

Applying the GDS Design Principles to the current design of the open registers platform yields some useful directions for simplification.

| Principle | Application |
| --- | --- |
| Start with needs (user needs not government needs) | Our custodian users need to see and manage their lists, as easily as possible and using any of a wide variety of hardware and software. To promote assurance, our custodians should have access to the raw data of a register and to feel confident that this data is held within a long-lived, stable, widely-used platform. Users of register data, especially service teams, should have access to authoritative registers within their normal development environments. |
| Do less | In keeping with our [architecture approach](https://governmenttechnology.blog.gov.uk/2016/07/05/introducing-our-architecture-approach-and-principles/), we should have a strong bias toward starting with simple, cloud-based tools rather than building our own things, as government is rarely unusual. Even when presented with two functionally equivalent options, the simpler solution with fewer moving parts and wider use of commodity platforms should be selected. In the case of Open Registers, we have a built a proprietary system to store, manage, expose and replicate our data, and I suggest that there are commodity platforms and formats we should first consider. |
| Design with data | Based on our conversations with potential and current adopters of our platform, we have learned that our platform is perceived as complex, which will be a barrier to adoption. |
| Do the hard work to make it simple | This is probably the hardest thing for us. |
| Iterate. Then iterate again. |  |
| This is for everyone |  |
| Understand context |  |
| Build digital services, not websites |  |
| Be consistent, not uniform |  |
| Make things open: it makes things better |  |



## Proposed definition of an open register

Existing definition:

> A Register is a collection of data held within the register database and 

New definition:

> A Register is an authoritative list that has been published as a file by a custodian in a public github repository. 


## File Format

The list that comprises the Register will follow these conventions:

* The list will be saved in a single UTF-8 encoded text file, or in a collection of files if there are a very large number of entries.
* The files will be tab-delimited and meet the W3C specification for the [CSV on the web](https://www.w3.org/TR/2015/REC-tabular-data-model-20151217/)
* The first column will be a key, ideally a non-proprietary value that can be stored and managed within other systems. If an ISO or other widely-recognised standard exists for this kind of entity, the standard should be used.
* Fields that link to another register will be named after the key of the linked register and use the CURIE-style syntax of “schools-eng:1234”.
* The special fields “start-date” and “end-date” will always be present. These dates indicate the day at which the entry became valid, if known, and the date at which the value became invalid or superseded, if known. The handling of these values should be respected by index services built on registers such that it is always possible to include a date as part of a query. For example, I could ask a register for all current values (with a start-date not set or in the past, and an end-date not set or in the future), or I could ask for all values that were valid at any particular date in the past. Being able to set and work with these attributes is an important part of how reference data evolves over time.



## Update scenarios

There are several obvious ways for registers to be updated securely, without the development of proprietary software:

* The custodian can log in and edit the list live on github.com, thus implicitly creating a new commit.

## Future Directions

The following scenarios would require further thinking about registers and their implementation:

* The need to charge for access to a public register would require additional controls at the index layer. The core register file format could still be used in this case, but any git-based repository would need to be private rather than public, and an additional software service would need to be developed to handle the payments and permission workflow.

## One more thing

I’d also like to note that the software developed thus far has supported not only a data model but the development of a business process around register design and custodianship which will be invaluable in getting our ideas and platforms widely used by government. Understanding the technical decisions around inspectable logs and versioning has helped to drive our understanding of our security needs, and the development of the external representation, in particular, has pointed us toward what is needed within the index service layer. 

This is all valuable design and learning, developed collaboratively with users, and is clearly worth the investment and time put in thus far. This proposal for refactoring should not in any way take away from the work that has been done — I simply feel that we can now iterate based on what we have learned, and as is frequent in technology projects, our next version can be better by being simpler and more open.
