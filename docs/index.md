# Welcome to Dash+

Dash+ is a declarative behavioural modelling language for software-based systems, meaning a model in Dash describes a transition system.  It combines Statecharts-like hierarchical and concurrent control states with the Alloy language for describing guards and actions on individual transitions.  It is an extension of the Alloy language therefore statements in pure Alloy can be combined with the description of the transition system in Dash+.

## Tool Support

Verification tool support (model checking and model instance generation) for Dash+ has been created as an extension to the Alloy Analyzer.  It can be found at: [Dash+ fork of the Alloy Analyzer](https://github.com/WatForm/org.alloytools.alloy)

We also have a simulator for models of Alloy transition systems:
[ALDB](https://github.com/WatForm/aldb)

## Language Description

Dash extends Alloy with keywords to describe transitions in a hierarchical, concurrent state machines (similar to Statecharts).  Here is a small example:
```
sig Value {}

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

More details can be found in: 
* The above example is available in the file sample.dsh in the root directory.
* Jose Serna. Dash: Declarative Behavioural Modelling in Alloy. MMath thesis, University of Waterloo, David R. Cheriton School of Computer Science, 2019. [https://cs.uwaterloo.ca/~nday/pdf/theses/2019-01-jserna-mmath-thesis.pdf]


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

## Credits and Support

Dash+ was created by Jose Serna and Nancy Day.  It is currently supported by Tamjid Hossain (t7hossain@uwaterloo.ca) and Nancy Day (nday@uwaterloo.ca). Issues with respect to Dash's tool support can be posted on the [Issues page](https://github.com/WatForm/org.alloytools.alloy/issues).  Discussions regarding the language and tool support can be posted on the [Discussions page](https://github.com/WatForm/org.alloytools.alloy/discussions).

