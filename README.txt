Abstract Application

A function block (briefly FB) is bascially an i/o or mealy
automata. It has a finite set of states, finite sets of
input/output ports, and a finite set of conditional/guarded
transitions tr = (st0, st1, cond, outeffs). A transition tr
is enabled by a set of inputs (messages on some input ports)
if this set satisfied the transitions condition and the
function block state is the transition starting state, st0.
In this case, tr can fire, moving the function block to st1
and emiting outeffs (messages on its output ports). For each
port there is a finite set of signals/messages that can be
received/sent. The states, ports, and transitions form a
function block class. There may be several occurrences of a
given class, thus each occurrence is given an identifier as
well as a class identifier.


An Application (briefly App) is defined by a set of function
blocks and links between output ports and target ports (which
may be function block inputs, or app outputs). Since
links model `wires' between physical ports we assume the link
map is 1-1.

An application state consists of the states of its function 
blocks, together with its input/output message pools.
The initial state has all function blocks in their initial
states and an initial message (set) in the application 
input pool.

An App executes its function blocks in lock step (abstracting 
a timed/clocked system).  Given an application input pool,
every function block fires an enabled transition if its
set of transitions enabled by the current input is non-empty.
Then the outeffs from each fb flow through the app links
to become inputs, and the process repeats.
To ensure that an FB fires at most on transition
per round, each FB is given a boolean `ticked' attribute,
initially `false', which is set to `true' when a transition
fires, and reset to `false' when the outputs are collected.

From the initial state we can execute (rewrite) a number of
steps to watch the message flow. We can also search for bad
(or good) states to determine of the app behaves as desired.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Intruder model

The above execution model assumes the app is isolated.
Suppose it is open. What about intruders?  We consider a bounded intruder model, in which the intrude can inject
a finite number of messages (at arbitrary times).
To model all possible attacks, the intruder is
give a set `allMessages' of all possible messages 
of the given application, and a bound n.  For up to n
times the intruder can pick a message from allMessages
and inject it into the network.  Search will explore
injection of all messages at all possible time.
We call this the concrete intruder model.
To reduce the search space, the set of concrete messages 
coupled with a bound n, is replaced by n distinct symbolic
messages.  Each symbolic message can in principle be instantiated to any element of AllMessagers.
In practice symbols are instantiated opportunistically
to values that enable transitions.
Again search can be used to enumerate all attacks, i.e.
intruder message instances that when injected drive the
application to a bad state.  Note that applications either
get stuck or are (eventually) period, due to the finiteness
of the function blocks. Thus, because Maude's search does
loop detection, the search for all attacks is finite.

Let p be the path (states and transitions) to a bad state.
bad(p) is the set of instantiated intruder messages
delivered in p (in the concrete or symbolic case).

Then the attack set for a given application, badState
predicate and intruder bound, badM(A,badState,n) 
(implemented as getBadEvents) 
is the set of sets bad(p) for p a path from the inital 
state of A to badState.


Intruder Theorem.  
Let 
AS = [A,smsg1 ... smsg_n] be an application A with
symbolic intruder of bound n.

AC = [A,(AllMsgs,n)] be an application A with
concrete intruder of bound n. 

Then

msgs in bad(AS1,badState,n) iff msgs in bad(AC1,badState,n)

In particular the symbolic match generation (instantiation)
semantics is sound and complete. I.e. it generates all and
only valid attacks.

In fact, for each execution trace from AS
there is a corresponding trace from AC and conversely
for each execution trace from AC there is a corresponding
trace from AS.   The
corresponding states have the same function block states
and corresponding incoming messages.  Corresponding
execution rules deliver the same messages, gather the
same outputs, and inject corresponding intruder messages.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Deployment 

A System is a deployed application--a collection
of networked devices each one hosting one or more of the
applications function blocks.  
A device is modeled as an application with input/output
ports corresponding to external inputs/outputs of its
function blocks.
In addition to the set of devices, a System also has
pools of input and output messages.
A system state consists of the states of its devices
together with its message pools.

Execution of a system proceeds by delivering input messages
to target devices, then executing the application semantics
of devices for one round, then collecting messages from
devices input pools targeted for function blocks on other
devices, and storing them in the system input pool. Repeat.

The message flow semantics leaves messages in a devices
input pool that are targeted to local function blocks.
Also, since a device has no ports for messages to
its function blocks that are locally generated, 
no such messages can be delivered by the delivery process.

Deploying an application is essentially a theory
transformation. The function deployApp takes an application
and a mapping from function blocks to devices and returns a
system that is the deployed version of the application
corresponding to the mapping.

The module specifying the structure and execution rules
for systems simply adds the constructor for systems
that encapsulates the devices, and rules for the extra
layer of message semantics (delivering and collecting).


Deployment Theorem:

Let A be an application and S be a deployment.
Then
  Executions of A and S are bisimilar.
In particular the underlying function block transitions are are the same.

Furthermore we can add an intruder to a system.
We just consider concrete messages - a set of n messages
represents a potential attack instance.
If the deployment is 1 function block per device then

  [A,imsgs] is bisimilar [S,imsgs]

otherwise

  If [S,imsgs] leads to a bad state, then [A,imsgs] leads to
  (the same) badState.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Wrapping

A device protects communications among its function blocks,
hence the second part of the theorem above.  To protect
communications between function blocks on different devices
we transform a device into a wrapped device that protects
communications by signing specified messages and checking 
signatures.   
      wrap(sys,msgs)
Given a set of messages to protect, the application level
links, and the deployment map, input and output policies
are added to each device as follows.

      wrap(sys,msgs)  

(the application links and deployment map are implicit in sys)

Let M be a message to be protected, its destination a port on
on a function block on device d1, its source a port on
a function block on device d2 (determined using links and
the deployment map).  Then a policy rule is added to the
output policy of d2 to sign the (contents of) messages
from the source port, and a policy rule is added to the
input police of d1 to only accept messages to the destination
port if signed by d2.


Wrapper Theorem

  If msgs contains bad(App,badState,n) then
badState is not reachable from 
   [wrap(deploy(app,map),msgs),(allmsgs,n)]
  

Finding a suitable deployment map

Signing is expensive. The wrapper policy generation attempts
to minimize signing, but ignores the content of messages, so
could be more efficient, relative to a given deployment.
Deploying all function blocks on one device would eliminate
the need for signing, however it is generally not realistic.
Devices may have limited capacity. Also, in the presence of
time constraints and parallelism amongst function blocks,
this would force sequentializing function block transitions
and increasing execition cycle time. The function .... uses
information about execution time of function blocks on
proposed devices, dependencies between function blocks
(represented by the application links) and timing
requirements to generate constraints on deployment maps.
These are exported to an SMT solver (Yices, Z3 ...) asking
for a model (in the case the constraints are satisfiable).
Thus we can find solutions that balance the need for
distribution and the cost of signing.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%zApplication structure

The Maude representation of a function block (FB)

  [fbId : fbCid |  fbAttrs] 

where fbId is the FBs identifier, fbCid its class identifier
and fbAttrs is a set of attribute - value pairs. These
include state : st ; oEvEffs : oeffs ; ticked : b where st
is the current state, oeffs a set of outputs (out :~ ev)
where out is an output port and ev the event, and b is a
boolean indicating whether the FB has fired a transition in
the current cycle.

A transition has the form tr(st0,st1,cond,oeffs) where
st0 is the initial state and st1 the final state, cond is
the condition, and oeffs is the set of outputs.
A condition is a boolean combination of primitive conditions
(in is ev) specifiying a particular event (ev) at input port in.   

Each function block class has a finite set of input/output
ports and a finite set of possible events at each port.
The set of states is also finite.

A function trsFB(fbCid) returns the set of transitions for function blocks of class fbCid.
trsFB(fbCid,st) selects the transitions in trs(fbCid) with
initial state st.

We compile a condition into a representation as a set of
constraint sets (CSetSet) which simplifies satisfaction
checking, and matching when messages are symbolic.

We can think of a constraint set (CSet) as a finite map from
function block inputs to finite sets of events.
A set of messages emsgs = {{{fbId,in_i},ev_k} | 1 <= i <= k}
satisfies a CSet, css, just if css has
size k,  the in_i form a set (one message per input) and ev_i in css(in_i) for 1 <= i <= k.

The function condToCSet(cond) returns a CSetSet
such that a message set emsgs satisfies some CSet in
in the result just if it statisfies cond.

This is lifted to transition the function
tr2symtr(tr(st1,st2,cond,oeffs) =
      symtr(st1,st2,condToCSet(cond),oeffs) . 

An application encapsulates a set of function blocks 
in a structure of the form

   [appId | appAttrs]

appAttrs is a set of attribute-value pairs including

   (fbs : funBs) and (iEMsgs : emsgs)

where funBs is a set of function blocks with unique
identifiers, and emsgs is the set of incoming messages of
the form {{fbId,in},ev}.

Links of the form {{fbId0,out},{fbId1,in}} connect ouputs of
one FB to inputs of another FB (possibly the same). They
also connect any application level inputs to FB inputs and
external FB outputs to application level outputs. In a well
formed application, each FB input has exactly one incoming
link and each FB output has exactly one outgoing link.

In principle the link set is an attribute of the application
structure.  In practice, since it models fixed `wires' connecting ports and does not change, to avoid redundant
information in traces, we specify a function appLinks(appId)
which is defined separately for each application
(in an application specific scenario module).

There are two execution rules for applications and
two rules modeling bounded intruder actions, one for the
concrete case and one for the symbolic case.

crl[app-exe1] fires a function block transition
if any are enabled. 

crl[app-exe1]:
 [appId | 
   fbs : ([fbId : fbCid | (state : st) ; (ticked : false) ;
            oEvEffs : none ;  fbAttrs] fbs1) ;
   iEMsgs : (emsgs0 iemsgs) ;
   ssbs : ssbs0 ;
   appAttrs ]
 =>
 [appId | 
   fbs : ([fbId : fbCid | (state : st1) ; (ticked : true) ;
             oEvEffs : oeffs ;  fbAttrs] 
           fbs1) ;
   iEMsgs : iemsgs) ;
   ssbs : (ssbs0 ssbs1) ;
   appAttrs ]
  if symtr(st, st1,[css] csss,oeffs) symtrs 
      := symtrsFB(fbCid,st)
  /\ size(emsgs0) = size(css)
  /\({ssbs1} ssbss) := genSol1(fbId,emsgs0,css)
.

The function genSol1(fbId,emsgs0,css) returns a set of
substitutions, consisting all and only substitutions
that match emsgs0 to a solution of CSet, css.
In the case of concrete messages,
the function genSol1 just returns an empty substitution
if emsgs0 satisfies css.

When rewriting, just one partition of iemsgs, one choice
of (symbolic) transition, and one satisfying substitution
is selected.  Search will explore all possible choices.

When [crl-exe1] is no longer applicable (not hasSol(fbs,iemsgs), crl[app-exe2] collects and route
output and prepares for the next cycle.

crl[app-exe2]:
 [appId | 
   fbs : fbs ; 
   iEMsgs : iemsgs ;
   oEMsgs : oemsgs ; 
   ssbs : ssbs  ; attrs]
 =>
 [appId | 
   fbs : fbs2 ; 
   iEMsgs : emsgs0 ;
   oEMsgs : (oemsgs emsgs1) ; 
   ssbs : ssbs  ; attrs1]
 if not hasSol(fbs,iemsgs)
 /\ tick := notApp(attrs)
 /\ not getTicked(attrs)  --- avoid extracting when no trans
 /\ attrs1 := setTicked(attrs, true)
 /\ {fbs2,emsgs0,emsgs1} := 
    extractOutMsgs(tick,fbs,none, 
                   none,none,appLinks(appId))  .
The function extractOutMsgs
collects outputs from the function blocks that fired
(and replaces oEFfEvs value with none)
routes them using appLinks(appId) to the 
linked FB input or application output.
Inputs are accumulated in emsgs0 and outputs
are accumulated in emsgs1.
The ticked attribute of each FB is set to the value of tick.
In the case of a basic application, this will be false
indicating the FB is ready for its next input.
When the application level execution rules are used
in a larger context, (notApp(attrs)) the function blocks
remain ticked, allowing further message routing
before repeating the execution cycle.  fbs2 collects the
updated function blocks.
If the application has a ticked attribute, it is set to
true, to indicate it has completed its cycle.

An application in the context of an intruder is represented
in the concrete case by a structure 
  [ app, emsgs, n]
where emsgs is a set of specific messages (typically all the
messages that could be delivered) and n is the number of
injection actions remaining.
The rule rl[app-intruder-concrete] selects one of the candidate messages, injects it, and decrements the counter.

rl[app-intruder-concrete]:
[[appId | fbs : fbs ; iEMsgs : emsgs0 ; 
 attrs], emsg emsgs, s n]
=> 
[[appId | fbs : fbs ; iEMsgs : (emsgs0 emsg) ;
    attrs], emsg emsgs, n] .

In the symbolic case an application open to intruders is
has the form
  [ app, emsgs]
where emsgs are symbolic messages of the
form {{idSym,inSym},evSym}.   We require different
messages to have distinct symbols.
If rule rl[app-intruder] selects one of the intruder messages
(if any are left), removes it from the intruder messages
and adds it to the incoming messages iEMsgs.

rl[app-intruder]:
[[appId | fbs : fbs ; iEMsgs : emsgs0 ;  attrs], 
 emsg emsgs]
=> 
[[appId | fbs : fbs ; iEMsgs : (emsgs0 emsg) ;
    attrs], emsgs] .

Using genSol1, such a symbolic message can be instantiated
to any deliverable message.   
Note that if a message is injected after all function blocks
have been ticked and before app-exe2 is applied, it will be
dropped by app-exe2, since function block inputs are cleared
before collecting the next round of inputs.
Note also, that concrete messages can be used in the intruder
set.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

Proof of intruder theorem

Define a correspondence between symbolic and concrete
intruder states as follows

 [As,smsgs] ~ [Ac,cmsgs,n]

if size(smsgs)  = n
   As.fbs   = Ac.fbs
   (As.iEMsgs)[ssbs] = Ac.iEMsgs for some symbol substitution

where for applications A, A.tag is the value of the attribute
with name `tag'.
(Note the attributes ssb and oEMsgs do not affect rule application.)

An execution trace is an alternating sequence of
(application) states and rule instances connecting adjacent
states as usual. A symbolic trace TrS and a concrete trace
TrC correspond if they have the same length and if the ith
elements correspond (for states as above and for rules,
instances of the same rule).

Theorem:  Let [As,smsgs] ~ [Ac,cmsgs,n] be inital application
states -- no intruder messages injected.

If TrC is an execution trace starting with [Ac,cmsgs,n]
then there is a corresponding execution trace
starting with [As,smsgs] and conversely.

Proof. Buy induction on trace length.
The base case is simple in either direction,
since an intruder message is only involved 
if the rule is an app-intruder rule.

Consider a TrS = TrS_0 -> [As_k,smsgs_k] -rl_k->
[As_k+1,smsgs_k+1]. By induction, let TrC_0(pmsgs)->
[Ac_k,cmsgs_k,n_k] be the set of corresponding concrete
traces where pmsgs are parameters for choicesof injected
concrete messages that remain in iEMsgs (have been injected
and not delivered or cleared), thus were injected since the
last app-exe2 rule.

If rl_k is an instance of crl[app-exe1] then As_k.iEMsgs =
iemsgs = iemsgs0 emsgs0 and function block fbId has a
transition delivering emsgs0[ssbs]. Claim for some choice of
a subset of parameters pmsgs0 as msgs0 Ac_k.iEMsgs =
iemsgs0' emsgs0[ssbs] where iemsgs0' ~ iemsgs0 with
parameters matched to remaining symbolic messages, if any.
Thus
    [Ac_k[pmsgs1,msgs0],cmsgs_k,n_k]
      -app-exe1> 
    [Ac_k+1[pmsgs1],cmsgs_k,n_k] ~ [As_k+1,smsgs_k]

For rl_k an instance of crl[app-exe2] or the intruder rule
it is easy to see that TrC extends to a corresponding trace.
[[qed -- more or less]]

Conversely, let TrC = TrC_0 -> [Ac_k,cmsgs_k,n_k] -rl_k-> [Ac_k+1,cmsgs_k+1,n_k+1] be a concrete trace.
Let TrS_0->[As_k,smsgs_k] be a corresponding symbolic trace. 
if rl_k is an instance of crl[app-exe1] then Ac_k.iEMsgs =
iemsgs = iemsgs0 emsgs0 and function block fbId has a
transition delivering emsgs0. 
Let ssbs be a substitution such that As_k.iEMsgs =
iemsgs' = iemsgs0' emsgs0' and emsgs0'[ssbs] = emsgs0.
By `completeness' of genSym1 ssbss will be a solution
for an As_k transition delivering emsgs0'[ssbs].
Thus 
   [As_k,smsgs_k] -rl_k->
   [As_k+1,smsgs_k] ~ [Ac_k+1,cmsgs_k+1,n_k+1] 
extending TrS_0 to TrS corresponding to TrC.

If rl_k is an instance of crl[app-exe1] or an intruder rule
it is easy to see that TrS_0 extends as desired.


Corrollary search enumerates all bad messages.
TO DO --CLEANUP
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Deployment

A deployed application has sort System and is represented in Maude by structures of the form:

  [sysId | appId | sysAttrs]
  
where sysAttrs is a set of attribute-value pairs including

   (devs : devs) and (iMsgs : msgs)

where devs is a set of devices, where a device has
the structure of an applications with additional attributes
including (ticked : b).
The function blocks of the application named by appId
are distributed amongst the devices.
The function sysMap maps FB identifiers to the identifier
of the device where the FB is hosted.
Each device has incoming/outgoing ports corresponding to 
links between its function blocks and function blocks on
other devices.   

The function deployApp(sysId,A,sysMap(sysId))
produces the deployment of application A as
a system with identifier sysId and
FBs distributed to devices according to sysMap(sysId).

  ceq deployApp(sysId,app,smap) =
         mkSys(sysId,getId(app),devs,msgs)  
  if emsgs := getIEMsgs(app)
  /\ devs := deployFBs(getFBs(app),none,smap)
  /\ msgs := emsgs2imsgs(sysId,emsgs,smap,none) .

The execution rules for applications apply to devices.
There are two additional rules for system execution:
crl[sys-deliver] and  crl[sys-collect].

The rule crl[sys-deliver] delivers each message associated to
the iMsgs attribute to the device containing the target
function block, and then sets the ticked attributes to
false enabling the application level rules.

  crl[sys-deliver]:
   [sysId | appId | devs : devs ; iMsgs : imsgs ; attrs]
  =>
   [sysId | appId | devs : devs1 ; iMsgs : none ; attrs]
  if isDone(devs) 
  /\ imsgs =/= none
  /\ devs1 := 
      deliver2Devs(devs,imsgs, 
                   appLinks(appId),sysMap(sysId)) .
isDone(devs) checks that all the devices have ticked
attribute set to true.                  

The rule crl[sys-collect] collects and distrubes messages
produced by the app level execution.

 crl[sys-collect]:
   [sysId | appId |
     devs : devs ; 
     iMsgs : imsgs ; 
     oMsgs : omsgs ; 
     attrs]
  =>
   [sysId | appId |
     devs : devs2 ; 
     iMsgs : (imsgs imsgs1) ; 
     oMsgs : (omsgs omsgs1) ; 
     attrs]
  if isDone(devs) 
  /\ {devs1, imsgs1, omsgs1} := 
        extractOutMsgs(sysId,devs,none,none,none,
                       appLinks(appId),
                       sysMap(sysId))
  /\ devs2 := (if (imsgs1 == none) --- only internal msgs
               then resetTicks(devs1,none)
               else devs1  --- ticks will be reset by deliver
               fi)  .
extractOutMsgs collect message from
device oEMsgs attributes (setting the attribute to none).
The updated devices are accumulated in devs1, messages
routed to FBs in some device are accumulated in imsgs1
and messages destined somewhere outside the system are
accumulated in omsgs1.


Deployment Theorem:

Let A be an application and S be a deployment.
Then A and S have corresponding executions.
In particular the underlying function block transitions are are the same.  Thus properties that are based only on
FB states are preserved.

Proof.

let sys-exe be the composite rule
     app-exe2;sys-collect;sys-deliver
The corresponding executions are obtained by
replacing app-exe2 in the application exection
by sys-exe in the system execution.


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Intruders at the system level


    
rl[sys-intruder]:
[[sysId | appId | 
   devs : devs ; 
   iMsgs : imsgs ;
   oMsgs : omsgs ; 
   attrs], msg msgs]
=> 
[[sysId | appId | 
   devs : devs ; 
   iMsgs : (imsgs msg) ;
   oMsgs : omsgs ; 
   attrs], msgs] .   



