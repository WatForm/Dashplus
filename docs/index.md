# Welcome to Dash+

Dash+ is a declarative behavioural modelling language for software-based systems, meaning a model in Dash describes a transition system.  
Dash is an extension of Alloy. It combines the control-oriented constructs of Statecharts with the 
declarative modelling of Alloy.
From Statecharts, Dash inherits a means to specify hierarchy, concurrency, and 
communication, three useful aspects to describe the behaviour of reactive systems.
From Alloy, Dash uses the expressiveness of relational logic and set theory to 
abstractly and declaratively describe structures, data, and operations.
It is an extension of the Alloy language therefore statements in pure Alloy can be combined with the description of the transition system in Dash+. 

The meaning of the name is 'Declarative Abstract State Hierarchy'.  The '+' was added when we added replicated concurrent control states.

These web pages are under development!

## Tool Support

Verification tool support (model checking and model instance generation) for Dash+ has been created as an extension to the Alloy Analyzer.  It can be found at: [Dash+ fork of the Alloy Analyzer](https://github.com/WatForm/org.alloytools.alloy)

We also have a simulator for models of Alloy transition systems:
[ALDB](https://github.com/WatForm/aldb)

## Language Description

Dash extends Alloy with keywords to describe transitions in a hierarchical, concurrent state machines (similar to Statecharts).  Here is a small example:
```
sig Value {} // regular Alloy declaration

conc state Root { // root state is always concurrent
	event ev1 {} 
	event ev2 {}
	v1,v2: Value  // variables declared within a state are dynamic

	conc state A { 
		default state A1 {
			trans t1 {
				from A1 // src state
				goto A2  // dest state
				on ev1 // guarding event
				when v2 = v1 // guarding condition: Alloy formula
				do {
					v1' = v1  // Alloy formula
					// refer to next value of a variable using primed variable
				}
				send ev2 // event sent
			}
		}
		state A2 {
			// parts of the transition may be omitted
			trans t2 {
				goto A1
				on ev2
				do {
					v2' = v2
				}
				send ev1
			}
		}
	}
	conc state B {
		trans t3 {
			on ev2
			do {
				v1' = v2
			}
		}
	}
}
```


### Semantics

A Dash+ model is a transition system meaning it the changing behaviour of variables over time/executions/traces.  A Dash+ model consists of 1) a Snapshot, which is a set of variables whose values change over time and 2) a set of transitions, each of which can cause some changes to the set of variables.  There can also be constants whose values do not change over time (described in pure Alloy).  

We can translate a Dash model into a CoreDash model, which completely separates the state hierarchy from the transitions and uses fully qualified names in the transitions.

* choosing a transition in each step
* priority of transitions
* stuttering

* Invariants states within a state holds whenever the model is in that control state.

### Name Spaces

```
conc state Bit1 {
	default state Bit1 {}
}
```

### Syntactic Sugar

* Transition Comprehension
	- scope of *

* Transition Defintions



## Writing Properties in Alloy

To write properties of Dash models to run/check, it is important to know a little bit about how Dash is translated to Alloy:
- the signature of system states is called 'Snapshot' 
- elements of the transition system are prefixed by the sequence of labelled parent states, e.g., for the above model
	+ Root_A_A1 is basic state A1 (which can be compared to s.conf where s is a Snapshot to see if A1 is in the set of states of this Snapshot)
	+ Root_A_A1_t1 is transition t1 (which can be compared with s.taken where s is a Snapshot to see if t1 is in the set of transitions taken so far in this big step) 
	+ Root_A_v1 is the value of variable v1 
	+ Root_ev1 is ev1, which can be compared to s.events where s is a Snapshot
- Dash creates predicates (**NOT** relations) called 'init[s:Snapshot]' and 'small_step[s,s_next: Snapshot]', which are the model.  A user can relate these predicates to other relations depending on the properties they want to check:
	+ for trace-based property checking (an option that can be chosen), a fact connecting 'init[]' to set 'first' and 'small_step[]' to relation 'next' in the ordering module is created
	+ for transitive-closure-based model checking (TCMC) (an option that can be chosen), a fact connecting 'init[]' to set 'ks_s0' and 'small_step[]' to relation 'ks_sigma is' added
	+ to just use the transitive closure, apply the transitive closure to 'next' from the ordering module or create a separate relation (connected to small_step) to avoid using the ordering module
	+ in the future, we plan to create output for Electrum-base model checking
- to only check the property at stable snapshots, use the boolean s.stable where s is a snapshot (stable is only relevant if there are concurrent states)
- if you need a scope for the EventLabel, count the number of events declared in the model. If you need a scope for the StateLabel, count the number of basic states declared in the model.

## Well-formedness Constraints

* The source and destination of a transition must be an OR-state or a basic state.
* An assignment to an environmental variable or environmental events is not allowed.
* At a particular level in the hierarchy all states must be either OR states or AND states.
* Primed variables can only be used in actions (not guards).
* At each level in the hierarchy that consists of OR-states and basic states, there must be a default state.

## Optimizations in the Translation to Alloy

* If there are no events used in the model, the signatures and snapshot relations for events are not included.
* If the model does not have any AND-states, then every snapshot is stable, which simplifies many predicate definitions in the Alloy output from the translation and the stable flag is removed from the snapshot relations.

## Credits and Support

The Dash language was created by Jose Serna and Nancy Day. The integration of Dash within the Alloy Analyzer was completed by Tamjid Hossain.  Kai Hsiang Yang contributed to the integration with the GUI.  Nancy Day provided guidance on the implementation.  Information on Dash can be found in:

* Jose Serna, Nancy A. Day, and Shahram Esmaeilsabzali. Dash: Declarative behavioural modelling in Alloy with control state hierarchy. Journal of Software and Systems Modelling, Accepted Apr 2022.

* Jose Serna. Dash: Declarative Behavioural Modelling in Alloy. MMath thesis, University of Waterloo, David R. Cheriton School of Computer Science, 2019. [https://cs.uwaterloo.ca/~nday/pdf/theses/2019-01-jserna-mmath-thesis.pdf]

* Jose Serna, Nancy A. Day, and Sabria Farheen. Dash: A new language for declarative behavioural requirements with control state hierarchy. In International Workshop on Model-Driven Requirements Engineering (MoDRE) @ IEEE International Requirements Engineering Conference (RE), pages 64--68. IEEE Computer Society, September 2017. [https://cs.uwaterloo.ca/~nday/pdf/refereed/2017-SeDaFa-modre.pdf]

* Amin Bandali. A Comprehensive Study of Declarative Modelling Languages. MMath thesis, University of Waterloo, David R. Cheriton School of Computer Science, 2020. [https://cs.uwaterloo.ca/~nday/pdf/theses/2020-07-15-bandali-mmath-thesis.pdf]

* Ali Abbassi, Amin Bandali, Nancy A. Day, and Jose Serna. A comparison of the declarative modelling languages B, Dash, and TLA+. In International Workshop on Model-Driven Requirements Engineering (MoDRE) @ IEEE International Requirements Engineering Conference (RE), pages 11--20. IEEE Computer Society, August 2018. [https://cs.uwaterloo.ca/~nday/pdf/refereed/2018-AbBa-modre.pdf]

Dash continues to be developed.  Extensions to Dash are discussed in:

* Tamjid Hossain and Nancy A. Day. Dash+: Extending alloy with hierarchical states and replicated processes for modelling transition systems. In International Workshop on Model-Driven Requirements Engineering (MoDRE) @ IEEE International Requirements Engineering Conference (RE). IEEE, September 2021. [https://cs.uwaterloo.ca/~nday/pdf/refereed/2021-HoDa-modre.pdf]

## Support

Dash+ is currently supported by Tamjid Hossain (t7hossain@uwaterloo.ca) and Nancy Day (nday@uwaterloo.ca). Issues with respect to Dash's tool support can be posted on the [Issues page](https://github.com/WatForm/org.alloytools.alloy/issues).  Discussions regarding the language and tool support can be posted on the [Discussions page](https://github.com/WatForm/org.alloytools.alloy/discussions).

