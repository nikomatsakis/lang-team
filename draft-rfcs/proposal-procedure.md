- Feature Name: (fill me in with a unique ident, `my_awesome_feature`)
- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

* Introduce a new step, the **proposal**, that is taken before an RFC is created. 
    * **Proposals** are created by opening a pull request on the lang-team repository, and they are designed to be relatively short and easy to write, with a focus on motivation and a de-emphasis on the details of the solution.
  * A Zulip topic is also created to discuss the proposal.
* If somebody in the lang team likes the proposal and has available bandwidth, they will opt to serve as a **liaison**. The liaison will work with you to see the proposal move forward.
  * Depending on their scope, a proposal may go forward in one of two ways:
    * For small changes, simply create a PR.
    * For larger changes that require an RFC -- or even multiple RFCs -- create a
      [project group] to jointly work on drafting it.
* To help in tracking what the lang-team is up to, as well as the set of bandwidth available for each member, we will track (and make publicly visible)
  * Active project groups, grouped by liasion
  * Topics of interest that each member may wish to pursue
  * A "short list" of proposals that the liaison liked and would like to pursue, but only after some other projects in their queue have completed
* Finally, if there is no activity on a proposal for a fixed amount of time, then it will be merged into a directory of "draft proposals".
  * Proposals from this directory can be resurrected later if there are new ideas, or if there is a liaison who has available bandwidth.
* In some cases, we may opt to actively decline a proposal, in which case we will give an explanation for why we do not wish to take that direction or try to solve that problem.

In order to transition to this system:

* We will tag all existing lang-team RFCs as "needs proposal" and block the threads.
* We encourage authors of all RFCs to create proposals.
* Presuming we are happy with the new system, at some point we will announce a time when existing "needs proposal" RFCs will be closed (or merged as draft proposals).

[project group]: https://github.com/rust-lang/rfcs/pull/2856

# Motivation
[motivation]: #motivation

The RFC system is one of Rust's great strengths, but we have found over time that there are also some shortcomings. The hope is that the "proposal" process will allow us to better focus our energies and create a more collaborative environment. Some of the benefits we foresee are as follows.

## More collaboration throughout the design process

Many people have reported that they have ideas that they would like to pursue, but that they find the RFC process intimidating. For example, they may lack the detailed knowledge of Rust required to complete the drafting of the RFC. 

Under the proposal system, people are able to bring ideas at an early stage and get feedback. If the idea is something we'd like to see happen, then a group of people can gather to push the RFC through to completion. The lang-team liaison can provide suggestions and keep the rest of the lang-team updated, so that if there are concerns, they are raised early.

## More focus on RFCs that reflect current priorities

The RFC repository currently contains far too many RFCs for anyone to keep up with -- especially members of the lang-team. Some of these RFCs are unlikely to be accepted, either because the ideas are too early stage, or because they don't align well with the project's current goals. Other RFCs may be very solid indeed, but they can be lost in the noise. 

We expect that after this change, the only RFCs that will be open are those that have a lang-team liaison and which have already seen some amount of iteration in a project group. This makes the RFC repository a good place to monitor for ideas that have traction.

Meanwhile, the lang-team proposals directory will still contain quite a mix of ideas. However, unlike the RFC repository, we do not intend to allow proposals to stay open indefinitely. Proposals that do not receive a liaison (or have active discussion) will be merged into a draft directory, possibly to be resurrected later (but, regardless, serving as "design notes" fur possible future reference).

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Hello! If you would like

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation


# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

# Prior art
[prior-art]: #prior-art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that rust sometimes intentionally diverges from common language features.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities
[future-possibilities]: #future-possibilities

Think about what the natural extension and evolution of your proposal would
be and how it would affect the language and project as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the project and language in your proposal.
Also consider how the this all fits into the roadmap for the project
and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information.
