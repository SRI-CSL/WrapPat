{Executive summary of the Application Framework}

{About Applications}


An Application (briefly App) consists of a set of function
blocks (FBs) and links between output and target ports
(which may be function block inputs, or application
outputs). A function block is a finite state automaton
similar to mealy machines in that it transforms a set of
inputs to a set of outputs. Transitions are guarded by
input conditions. An application state consists of the
states of its function blocks, together with contents of
its input/output message pools. The initial state has all
function blocks in their initial states and an initial
message (set) in the application input pool.
Function blocks have a class identifier to refer to the
fixed structure (states, inputs/outputs, transitions),
and instance identifiers to refer to specific instances.
Each application has an identifier used to refer to its
fixed structure and its state in an execution sequence.

An App executes its function blocks in cycles
where in each cycle, the input pool is effectively
delivered to function block inputs and each function block
fires one transition if possible.  The remaining inputs
are cleared,  the function block outputs are collected,
routed along the application links, and stored in the 
application input pool. 

From the initial state we can execute (rewrite) a number of
steps to watch the message flow. We can also search for bad
(or good) states to determine of the App behaves as desired.


%%%%%%%%%%%%%%%%%%%%%%%%%
{Bounded Intruder Model}

The above execution model assumes the app is isolated. What if
the application executes in an environment with intruders. We
consider a bounded intruder model, in which the intrude can
inject a finite number of messages (at arbitrary times).

To talk about attacks we need a notion of undesired
consequence. Thus we assume a predicate badState is defined
on App states. We further assume the predicate only depends
on the function block part of an application state. We
assume that no isolated execution of an application reaches
a bad state (application state satisfying for which
badState is true).

To model all possible attacks, the intruder is
given a set of all possible messages deliverable in
in the given application.  For up to n
times the intruder can pick a message from this set
and inject it into the the application input pool. 
We use search to determine which message injections
can lead to a bad state (a successful attack).  

We call this the concrete intruder model. Clearly the search
space grows rapidly with number of messages and bound. To
reduce the search space, the set of concrete messages coupled
with a bound n, is replaced by n distinct symbolic messages.
Each symbolic message can in principle be instantiated to any
deliverable measage. In practice symbols are instantiated
opportunistically to values that enable transitions at
delivery time. Again search can be used to enumerate
successful attacks.

Note that applications either
get stuck or are (eventually) period, due to the finiteness
of the function blocks. Thus, because Maude's search does
loop detection, the search for all attacks is finite.

Using the reflection and the search
descent functions, we define the function  
     \texttt{getBadEMsgs}
that enumerates the attacks (bad message sets) given
an application in a symbolic intruder environment.


Intruder Theorem.  This theorem  states that
each execution of an application  \texttt{A} 
in a symbolic intruder
environment has a corresponding execution of \texttt{A}  in
the concrete intruder environment with the same bound.
In corresponding executions (formally alternating sequences
of states and rule instances) corresponding states have the same function block states and corresponding incoming messages.  Corresponding execution rules deliver the same messages, gather the same outputs, and inject corresponding intruder messages.  Thus enumeration of attackes gives
the same result in both cases.
The key to this result is the soundness and completeness
of the symbolic match generation.


%%%%%%%%%%%%%%%%%%%%%%%
Deployment

Applications are abstract behavior models.  
As a step towards modeling additional features
we transform application level models to system
level models by deploying function blocks on
one or more devices.
A device is modeled as an application with input/output
ports corresponding to external inputs/outputs of its
function blocks.
In addition to the set of devices, a System also has
pools of input and output messages.

Execution of a system cycle proceeds by delivering input
messages to target devices, then executing the application
semantics of devices for one cycle, then collecting
messages from devices input pools that are targeted for
function blocks on other devices, and storing them in the
system input pool.

The message collection process leaves a message in the input pool of a device in that pool if
it is targeted to a function block on the same device.
The delivery process drops messages from the system input
pool that should flow along links internal to a device,
since there will be no device input ports for such messages.

Deploying an application is essentially a theory
transformation \cite{meseguer-meseguer-14facs-scp}. 

The modules specifying the structure and execution rules
for systems extends the corresponding application modules
with constructors for systems (and embedding systems into
intruder environments) and rules for the extra layer of
message semantics (delivering and collecting). The function

deployApp(sysId,A,idmap) 

takes an application A and a mapping from function blocks 
(identifiers) to devices (their identifiers) and returns a system (with identifier sysId) that is the deployed version
of the application corresponding to the mapping.

Deployment Theorem: This theorem states that executions of
an application A and a deployment S =
deployApp(sysId,A,idmap) of A are in close correspondence. In
particular the underlying function block transitions are
are the same.  

We add an intruder to a system's execution
environment in analogy to the applciation intruder
model.  It is sufficient to consider intruders with
a finite set of concrete messages to inject at will
as we show that all attacks can be found at the application
level.  We extend the deployment transformation by lifting
the badState predicate to systems, relying on the fact
that it depends only on the function block states.

Deployment Intruder Theorem: This theorem states that,
letting A,S be as in the Deployment Theorem, (1) for any
execution of [S,msgs] there is a corresponding execution of
[A,emsgs] (where emsgs are the application level versions
of msgs), preserving the underlying function block
transitions; and (2) for any execution of [A,emsgs] that does not deliver any intruder messages that should flow on links internal to some device, has a corresponding execution from [S,msgs].

Thus any attack at the system level has a
corresponding attack at the application level. 
The condition in part (2) is because internal messages are protected by the device execution semantics.
If the deployment map is 1-1 then there are no internal
messages, so all [A,emsgs] executions have a corresponding
system level execution.  


%%%%%%%%%%%%%%%%%%%%
Wrapping

A device protects communications among its function blocks,
hence the second part of the theorem above.  To protect
communications between function blocks on different devices
we transform a system S into a system, wrap(S,emsgs), in which devices are wrapped 
in a policy layer protecting
communications between devices by signing messages 
and checking signatures on flows defined by emsgs.

Wrapper Theorem
Let A be an application and emsgs a set of message containing
all elements of message sets in badMsgs(A,n),  then
badState is not reachable from 
   [wrap(deploy(sysId,A,idmap),emsgs),msgs]
for any intruder message set msgs of size at most n.   
  
%%%%%%%%%%%%%%%%%%%%%
Finding a suitable deployment map

Signing is expensive. The wrapper policy generation attempts
to minimize signing, but considers only endpoints and 
ignores the content of messages, so it
could be more efficient, relative to a given deployment.
Deploying all function blocks on one device would eliminate
the need for signing, however this is generally not realistic.
Devices may have limited capacity. Also, in the presence of
time constraints and parallelism amongst function blocks,
this would force sequentializing function block transitions
and increasing execition cycle time. The function mkDeployCstr 
uses information about execution time of function blocks on
proposed devices, dependencies between function blocks
(represented by the application links) and timing
requirements to generate constraints on deployment maps.
These are exported to an SMT solver (Yices, Z3 ...) asking
for a model (in the case the constraints are satisfiable).
Thus we can find solutions that balance the need for
distribution and the cost of signing.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Formalization of the I4.0 framework in Maude
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

We now  describe the formal representation  of Applications and
Systems in Maude,  give formal statements of  the theorems with
key  ideas for  the  proofs.  The Maude  code  is available  at
{\small\texttt{https://github.com/SRI-CSL/PickNPlace.git}}.
There you  will also find documentation,  scenarios, and sample
runs.
  
We describe the main structures, operations, and rules using
Maude notation, leaving sort and variable declarations implicit.


Function block structure

The Maude representation of a function block (FB) is a term
of the form

  [fbId : fbCid |  fbAttrs] 

where fbId is the FB identifier, fbCid its class identifier
and fbAttrs is a set of attribute-value pairs, including

  (state : st) ; (oEvEffs : oeffs) ; (ticked : b)

where state, oEvEffs, ticked are the attribute tags, st is the
current state, oeffs a set of signals/events to be
transmitted, and b is a boolean indicating whether the FB has
fired a transition in the current cycle. An element of oeffs
has the form (out :~ ev) (out is an output of the FB and ev
the event to be transmitted. We give FBs blocks both an FB
identifier and a class identifier to allow for multiple
occurrences of a given class.

A function block class is characterized by its finite set of
states, finite sets of inputs and outputs, a finite set of
possible events at each input or output, and a finite set of
conditional transitions. A transition has the form
tr(st0,st1,cond,oeffs) where st0 is the initial state and st1
the final state, cond is the condition, and oeffs is the set
of outputs. A condition is a boolean combination of primitive
conditions (in is ev) specifiying a particular event (ev) at
input . A transition tr(st0,st1,cond,oeffs) is enabled by a
set of inputs if they satisfy cond and the current state of
the function block state st0. In this case, tr can fire,
changing the function block state to st1 and emitting oeffs.

The function trsFB(fbCid) returns the set of transitions for function blocks of class fbCid.  The remaining feature (states,
etc., can be computed from the transitions.)
trsFB(fbCid,st) selects the transitions in trs(fbCid) with
initial state st.

We compile a transition condition into a representation as a
set of constraint sets (CSetSet) which simplifies satisfaction
checking, and matching when messages are symbolic.

We can think of a constraint set (CSet) as a finite map from
function block inputs to finite sets of events.
A set of inputs ieffs = {(in_i :~ev_k) | 1 <= i <= k}
satisfies a CSet, css, just if css has
size k,  the in_i form a set equal to the domain of css,
and ev_i  is in css(in_i) for 1 <= i <= k.

The function condToCSet(cond) returns a CSetSet
such that an input set ieffs satisfies some CSet in
in the result just if it statisfies cond.

This is lifted to transitions by the function
tr2symtr(tr(st1,st2,cond,oeffs) =
      symtr(st1,st2,condToCSet(cond),oeffs) . 

%%%%%%%%%%%%%%%%%%%%%%%
Application structure and semantics

An application encapsulates a set of function blocks 
in a structure of the form

   [appId | appAttrs] .

appAttrs is a set of attribute-value pairs including

   (fbs : funBs) and (iEMsgs : emsgs)

where funBs is a set of function blocks (with unique
identifiers), and emsgs is the set of incoming messages of
the form {{fbId,in},ev}.  

Notation.  We use fbId, fbId0 ... for FB identifiers,
in/out for FB input/output connections, and ev for the event/
signal transmitted by a message.  Terms of the form {fbId,in/out} are called Ports.  For entities X with attributes
we write X.tag for the value of the attribute of X
with name `tag'.

Links of the form {{fbId0,out},{fbId1,in}} connect ouput ports
of one FB to inputs of another FB (possibly the same). They
also connect application level inputs to FB inputs and FB
external outputs to application level outputs. In a well
formed application, each FB input has exactly one incoming
link and each FB output has exactly one outgoing link.

In principle the link set is an attribute of the application
structure.  In practice, since it models fixed `wires' connecting function block outputs and inputs and does not change, to avoid redundant
information in traces, we specify a function appLinks(appId)
which is defined separately for each application
(in an application instance specific scenario module).
Since links model `wires' between physical connectors we require the link map of  a well-formed application to be 1-1 and include every function block output port in its domain.


There are two execution rules for application behavior and
two rules modeling bounded intruder actions, one for the
concrete case and one for the symbolic case.

To ensure that an FB fires at most on transition
per round, each FB is given a boolean `ticked' attribute,
initially `false', which is set to `true' when a transition
fires, and reset to `false' when the outputs are collected.

The rule crl[app-exe1] fires a function block transition
if one is enabled and sets the ticked attribute to true
to ensure each FB fires at most one transition in a cycle.

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

When [crl-exe1] is no longer applicable (hasSol(fbs,iemsgs) is false), crl[app-exe2] collects and routes generated
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

The function extractOutMsgs emoves outputs from the function
blocks that fired routes them using appLinks(appId) to the
linked FB input or application output. Application level
inputs are accumulated in emsgs0 and outputs are accumulated
in emsgs1. The ticked attribute of each FB is set to the value
of tick. In the case of a basic application, this will be
false indicating the FB is ready for the next cycle. When the
application level execution rules are used in a larger
context, (notApp(attrs) is true), extractOutMsgs ensures that
each FBs ticked attribute is true, allowing further message
processing before repeating the execution cycle. fbs2 collects
the updated function blocks. If the application has a ticked
attribute, it is set to true, to indicate it has completed the
current cycle.

An application A in the context of an intruder is represented
in the concrete case by a structure of the form

  [A, emsgs, n]

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

An application A in the context of a symbolic intruder is represented by a structure of the form

  [ A, smsgs]
  
where smsgs are symbolic intruder messages of the
form {{idSym,inSym},evSym} (idSym, inSym, evSym are symbols
standing for function block identifiers, inputs, and events
respectively).   We require different
messages to have distinct symbols.

The rule rl[app-intruder] selects one of the intruder messages
(if any are left), removes it from the intruder message set
and adds it to the incoming messages iEMsgs.

rl[app-intruder]:
[[appId | fbs : fbs ; iEMsgs : emsgs0 ;  attrs], 
 emsg emsgs]
=> 
[[appId | fbs : fbs ; iEMsgs : (emsgs0 emsg) ;
    attrs], emsgs] .

We note that the rule works equally well with concrete or
symbolic messages, allowing one to explore consequences of 
injecting specific messages.
Using genSol1, a symbolic message can be instantiated
to any deliverable message.   

Note that if a message is injected after all function blocks
have been ticked and before app-exe2 is applied, it will be
dropped by app-exe2, since function block inputs are cleared
before collecting the next round of inputs.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

The intruder theorem.

We define a correspondence between symbolic and concrete
intruder states as follows

 [As,smsgs] ~ [Ac,cmsgs,n]

just if size(smsgs)  = n,  As.fbs = Ac.fbs, and
(As.iEMsgs)[ssbs] = Ac.iEMsgs for some symbol substitution ssbs.  
(Note that the attributes ssb and oEMsgs do not affect rule application.)

An execution trace is an alternating sequence of
(application) states and rule instances connecting adjacent
states as usual. A symbolic trace TrS  from [A,smsgs] and a concrete trace TrC from [A,emsgs,n] correspond if they have the same length and if the ith elements correspond.
Correspondence of state is the relation ~ above.
Two rule instances correspond if they are instances of
the same rule and in the app-exe1 case the instances
are the same transition of function blocks with the same
identifier.

Theorem: Let [A,smsgs] ~ [A,cmsgs,n] be inital application
states in symbolic and concrete intruder environments
respectively, where no intruder messages have been injected
and size(smsgs) = n.

If TrS is an execution trace starting with [A,smsgs]
then there is a corresponding execution trace
starting with [A,cmsgs n] and conversely.

Proof. Buy induction on trace length.
The base case is simple in either direction,
since an intruder message is only involved 
if the rule is an app-intruder rule.

Consider TrS = TrS_0 -> [As_k,smsgs_k] -rl_k->
[As_k+1,smsgs_k+1] and exeution trace from [A, smsgs].
By induction, let TrC_0(pmsgs)->
[Ac_k,cmsgs,n_k] be the set of corresponding concrete
traces where pmsgs are parameters for choicesof injected
concrete messages that remain in iEMsgs (have been injected
and not delivered or cleared), thus were injected since the
last app-exe2 rule.

If rl_k is an instance of crl[app-exe1] then As_k.iEMsgs =
iemsgs = iemsgs0 emsgs0 and function block fbId has a
transition delivering emsgs0[ssbs]. 
Let iemsgs0 = iemsgs00 iemsgs01 and emsgs0 = emsgs00 emsgs01
where iemsgs00, emsgs00 are concrete and iemsgs01, emsgs01
are symbolic.
Claim Ac_k.iEMsgs = iemsgs00 ipmsgs01 emsgs00 pmsgs01
where ipmsgs01 pmsgs01 are injection message parameters
with size(pmsgs01) = size(emsgs01) and
size(ipmsgs01) = size(iemsgs01).
Thus we choose pmsgs01 = emsgs01[ssbs] and
extend TrC by a corresponing application of crl[app-exe1]
to [Ac_k[pmsgs01 = emsgs01[ssbs]],cmsgs,n_k] ~ [A_k+1,pmsgs].

For rl_k an instance of crl[app-exe2] or the intruder rule
it is easy to see that TrC extends to a corresponding trace.

Conversely, let TrC = TrC_0 -> [Ac_k,cmsgs_k,n_k] -rl_k->
[Ac_k+1,cmsgs_k+1,n_k+1] be a concrete trace. By induction let
TrS_0->[As_k,smsgs_k] be a corresponding symbolic trace. if
rl_k is an instance of crl[app-exe1] then Ac_k.iEMsgs = iemsgs
= iemsgs0 emsgs0 and function block fbId has a transition
delivering emsgs0. Let ssbs be a substitution such that
As_k.iEMsgs = iemsgs' = iemsgs0' emsgs0' and emsgs0'[ssbs] =
emsgs0. By `completeness' of genSol1 ssbs will be a solution
for an fbId transition delivering emsgs0'[ssbs].
Thus 
   [As_k,smsgs_k] -rl_k->
   [As_k+1,smsgs_k] ~ [Ac_k+1,cmsgs_k+1,n_k+1] 
extending TrS_0 to TrS corresponding to TrC.

If rl_k is an instance of crl[app-exe2] or an intruder rule
it is easy to see that TrS_0 extends as desired.


Corollary search using the symbolic intruder model for paths reaching a badState finds all successful (bounded) attacks.

We define the function getBadEMsgs(moduleName,[A,smsgs])
that returns the set of injected message sets that
lead to badState.  
This function uses reflection to enumerate search paths
reflecting the command 

  search in moduleName : [A,smsgs] =>+ appInt:AppIntruder 
  such that badState(appInt:AppIntruder) .

Injected symbolic messages are determined by looking for
adjacent states where the symbolic message set decreases.
The symbols of injected messages that were actually delivered are in the domain of the value of the sbss attribute of the final state.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Deployment

A deployed application is called a System and is represented in Maude by terns of the form:

  [sysId | appId | sysAttrs]
  
where sysAttrs is a set of attribute-value pairs including

   (devs : devs) and (iMsgs : msgs)

devs is a set of devices, and msgs is a set of system level
messages of the form {srcPort,tgtPort,ev} where srcPort 
and tgtPort are terms of the {devId, {fbId, out}}
and {devId, {fbId, in}} respectively.

A device is represented as an application term with additional attributes including (ticked : b).
The function blocks of the application named by appId
are distributed amongst the devices.
The function sysMap(sysId) maps FB identifiers to the identifier
of the device where the FB is hosted.
Each device has incoming/outgoing ports corresponding to 
links between its function blocks and function blocks on
other devices.   

The function deployApp(sysId,A,sysMap(sysId))
produces the deployment of application A as
a system with identifier sysId and
FBs distributed to devices according to sysMap(sysId).

  ceq deployApp(sysId,app,idmap) =
         mkSys(sysId,getId(app),devs,msgs)  
  if emsgs := getIEMsgs(app)
  /\ devs := deployFBs(getFBs(app),none,idmap)
  /\ msgs := emsgs2imsgs(sysId,emsgs,idmap,none) .

The function deployFBs creates an empty device
for each device identifier in the range of idmap
(setting iMsgs to none and ticked to true).
Then each FB (identifier fbId) of app is added to the 
fbs attribute of the device identified by idmap[fbId].

Note that the deployApp function can be applied to any
state A_k in an execution trace from A. A system S_k can be
abstracted to an application by collecting all the device
function blocks in the application fbs attribute,
collecting the iEMsgs attributes of devices into the iEMsgs
attribute of the applicaton and converting system level
input messages to application level messages added to the
iEMsgs attribute of the application.


The execution rules for applications apply to devices in
a system.
There are two additional rules for system execution:
crl[sys-deliver] and  crl[sys-collect].

The rule crl[sys-deliver] delivers each message associated
to the iMsgs attribute to the device containing the target
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
attribute set to true.   The target port of an imsg identifies the device and function block for delivery.
The links and idmap are used to check if the device
has an open port.

The rule crl[sys-collect] collects and distributes messages
produced by the application level execution rules.

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
               else devs1 --- ticks will be reset by deliver
               fi)  .

The function extractOutMsgs collects message from
device oEMsgs attributes (setting the attribute to none),
converts them to system level messages and accumulates
them in omsgs1.  Messages from device iEMsgs attributes 
are split into local and external.  The local messages
are left, the external messages are converted
to system level messages and accumulated in imsgs1.
The updated devices are accumulated in devs1.

Deployment Theorem.

We define correspondence between execution traces from an
application A, and a deployment S = deployApp(A,idmap). 
A application state A' corresponds to a system state S'
just if they have the same function blocks and the same
undelivered (input) messages.  (Note that the deployment and abstraction operations are subsets of this correspondence relation.)  
An instance of crl[app-exe1]  in a application trace
corresponds to the same instance of that rule in a
system trace (fires the same transition for the same function block).   An instance of app-exe2 in a application
trace corresponds to a sequence app-exe2;sys-collect;sys-deliver in a system trace
collecting and delivering corresponding messages.


Theorem. Let A be an application and S =
deployApp(A,idmap) be a deployment of A

Then A and S have corresponding executions.

Proof.
This is a direct consequence of the definition of
corresponding traces.


Corollary.  A and S satisfy the same properties
that are based only on FB states and transitions.

This is because corresponding traces have the same underlying function block transitions.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Intruders at the system level

Deployed applications are embedded in an intruder
environment analogously to the abstract applications.
We consider a simple case where the intruder has
a finite set of concrete messages to inject,
and use that to show that any attack at the system level
can already be found at the application level.

Thus a system in a bounded intruder environment is a term of the form

    [sys,msgs]  
    
and we lift the deployment function to

deployAppI(sysId,[A,msgs],idmap) = 
[deployApp[sysMap,A,idmap],deployMsgs(msgs,appLinks(A),idmap)]

where deployMsgs transforms application level messages
{fbport,ev} to system level, {srcdevport,tgtdevport,ev}
using the link and deployment maps.

The intruder injection rule is lifted as follows:
    
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


System Intruder Theorem.

Assume Ai = [A,msgs] where A is an application in its initial
state (no intruder messages injected) and Si =
deployAppI(sysId,Ai,idmap).

The correspondence relation of the deployment theorem
extends naturally to the intruder case.  

1. If TrS is a trace from Si then there is a corresponding trace from Ai.  

2. If TrA is a trace from Ai that delivers no intruder messages
that flow on links internal to a device, then there is
a corresponding trace from Si.  

The proof is the same as for the correspondence of an
application and its deployment. The additional condition in
part 2 is needed because a device protects communications
between FBs it hosts by have no port for delivery of such
messages. In particular, if all the FBs are hosted on a single
device then no intruder messages can be delivered.

Corollary.
If a badState is reachable from Si then sys2app(msgs)
is an element of getBadEMsgs([A,smsgs]) where size(smsgs) =
size(msgs).

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Wrapping

Communications between function blocks on different devices
can be protected by standard cryptographic means such as
use of signatures.  Since we expect some devices to be resource
limited, and signing can be computationally expensive, we 
would like to avoid signing communications unnecessarily.
Towards this goal we define a further transformation
of applications:

   wrapApp(A,smsgs,idmap) 
    =
  wrapSys(deployApp(A,idmap),
         flatten(getBadEMsgs(mname,[A,smsgs])))

where flatten unions the sets in a set of sets.

wrapSys(S,emsgs) wraps the devices in S with policies
for signing and checking signatures of messages on flow
defined by emsgs.

A wrapped device has input/output policy attributes used to
control the flow of messages in and out of the device.
An inputoutput policy is an iFact/oFact set where an
iFact has the form [i : fbId ; in, devId] and an oFact has
the form [o : fbId ; out].

If [i : fbId ; in, devId] is in the input policy of a device then
a message {{fbId,in}, ev} is accepted by that device only if ev
is signed by devId, otherwise the message is dropped. Dually, if
[o : fbId ; out] is in the output policy of a device then when a
message {{fbId,out}, ev} is transmitted ev is signed by the
device.

Following the usual logical representation of crypto functions,
we represent a signed event by a term sg(ev,devId), assuming only
the device with identifier devId can produce such a signature,
and any device that knows the device identifier can check the
signature.   

The function wrap-sys([sysId | appId | attrs],emsgs) invokes 
wrap-dev(dev,emsgs,appLinks(appId),sysMap(sysId),none,none)
to wrap each of its devices.  The two nones are initial values
of the policy accumulators, ipol and opol.

  ceq wrap-dev(dev,{{fbId,in},ev} emsgs,links,idmap, 
               ipol,opol)
    = wrap-dev(dev,emsgs,links,idmap,
               (ipol ipol1), (opol opol1))
  if {{fbId0,out},{fbId,in}} links0 := links
  /\ devId1 := idmap[fbId]
  /\ devId0 := idmap[fbId0]
  /\ devId1 =/= devId0    ---- not an internal link
  /\ devId := getId(dev) 
  **** if emsg sent from dev add opol to sign outgoing
  /\ opol1 := (if devId == devId0 
               then [o : fbId0 ; out ]
               else none
               fi)
  **** if emsg rcvd by dev, require signed by sender devId0
  /\ ipol1 := (if devId == devId1
               then [i : fbId ; in, devId0]
               else none
               fi) . 

  eq wrap-dev(dev,emsgs,links,idmap,ipol,opol) =
      addAttr(dev,(iPol : ipol ; oPol : opol)) [owise] .
 

Wrapper Theorem

Assume A is an application, allEMsgs is the set of 
messages deliverable in an exeution of A,
and smsgs a set of symbolic messages of size n.
Assume badState is not reachable in an execution of A.

If emsgs contains flatten(getBadEMsgs([A,smsgs])) then

1. Let wA = [wrap(deployApp(A,idmap),emsgs)
 Then every execution from wA has a corresponding
 execution from A and conversely.  In particular
 badState is not reachable from wA.

2. badState is not reachable from 

   wAC = [wrap(deploy(A,idmap),emsgs),allEMsgs,n]

Proof 1.
Similar to the deployment theorem part 1, noting that by
definition of the wrap function, any message in emsg will be
signed by the sending device and thus will satisfy the receiving
device input policy and be delivered in the wA trace as it will in
the A trace.  
  
Proof 2.
Assume badState is reachable from wAC.
Let 
   wAC = wAC_0 rl_0 ..... rl_k wAC_k+1 

be a witness execution where badState holds of wAS_k+1.
By the assumption on A from part 1, at least one 
intruder message must have been delivered.

Let {emsg_1 ... emsg_l} be the intruder messages delivered
in the trace, say by rules rl_j_1 .. rl_j_l.  None of these
messages are in emsgs since their events cannot be signed
by one of the devices, and thus would not satisfy the 
relevant input policy.   
Thus there is a corresponding trace from the unwrapped system

   AC = [deploy(A,idmap),allMsgs,n]

and by the deploy intruder theorem there is a corresponding
trace from [A,allEMsgs,n] reaching a badState.
But emsgs contains all messages that are part of an intruder
message set which if injected can cause badState to be reached.
A contradiction.

