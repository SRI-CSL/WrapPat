{Executive summary of the Application Framework}

{About Applications}


An Application (briefly App) consists of a set of function
blocks (FBs) and links between output and target ports (which
may be function block inputs, or application outputs). A
function block is a finite state automaton similar to mealy
machines in that it transforms a set of inputs to a set of
outputs. Transitions are guarded by input conditions. An
application state consists of the states of its function
blocks, together with contents of its input/output message
pools. The initial state has all function blocks in their
initial states and an initial message (set) in the
application input pool. Function blocks have a class
identifier to refer to the fixed structure (states,
inputs/outputs, transitions), and instance identifiers to
refer to specific instances. Each application has an
identifier used to refer to its fixed structure and its state
in an execution sequence.

An App executes its function blocks in cycles where in each
cycle, the input pool is effectively delivered to function
block inputs and each function block fires one transition if
possible. The remaining inputs are cleared, the function
block outputs are collected, routed along the application
links, and stored in the application input pool.

From the initial state we can execute (rewrite) a number of
steps to watch the message flow. We can also search for bad
(or good) states to determine of the App behaves as desired.


%%%%%%%%%%%%%%%%%%%%%%%%%
{Bounded Intruder Model}

The above execution model assumes the app is isolated. What
if the application executes in an environment with intruders.
We consider a bounded intruder model, in which the intrude
can inject a finite number of messages (at arbitrary times).

To talk about attacks we need a notion of undesired
consequence. Thus we assume a predicate badState is defined
on App states. We further assume the predicate only depends
on the function block part of an application state. We assume
that no isolated execution of an application reaches a bad
state (application state satisfying for which badState is
true).

To model all possible attacks, the intruder is given a set of
all possible messages deliverable in in the given
application. For up to n times the intruder can pick a
message from this set and inject it into the the application
input pool. We use search to determine which message
injections can lead to a bad state (a successful attack).

We call this the concrete intruder model. Clearly the search
space grows rapidly with number of messages and bound. To
reduce the search space, the set of concrete messages coupled
with a bound n, is replaced by n distinct symbolic messages.
Each symbolic message can in principle be instantiated to any
deliverable measage. In practice symbols are instantiated
opportunistically to values that enable transitions at
delivery time. Again search can be used to enumerate
successful attacks.

Note that applications either get stuck or are (eventually)
period, due to the finiteness of the function blocks. Thus,
because Maude's search does loop detection, the search for
all attacks is finite.

Using the reflection and the search
descent functions, we define the function  
     getBadEMsgs
that enumerates the attacks (bad message sets) given
an application in a symbolic intruder environment.


Intruder Theorem.  
This theorem states that each execution of an application A
in a symbolic intruder environment has a corresponding
execution of A in the concrete intruder environment with the
same bound. In corresponding executions (formally alternating
sequences of states and rule instances) corresponding states
have the same function block states and corresponding
incoming messages. Corresponding execution rules deliver the
same messages, gather the same outputs, and inject
corresponding intruder messages. Thus enumeration of attackes
gives the same result in both cases. The key to this result
is the soundness and completeness of the symbolic match
generation.



%%%%%%%%%%%%%%%%%%%%%%%
Deployment

Applications are abstract behavior models. As a step towards
modeling additional features we transform application level
models to system level models by deploying function blocks on
one or more devices. A device is modeled as an application
with input/output ports corresponding to external
inputs/outputs of its function blocks. In addition to the set
of devices, a System also has pools of input and output
messages.

Execution of a system cycle proceeds by delivering input
messages to target devices, then executing the application
semantics of devices for one cycle, then collecting messages
from devices input pools that are targeted for function
blocks on other devices, and storing them in the system input
pool.

The message collection process leaves a message in the input
pool of a device in that pool if it is targeted to a function
block on the same device. The delivery process drops messages
from the system input pool that should flow along links
internal to a device, since there will be no device input
ports for such messages.

Deploying an application is essentially a theory
transformation \cite{meseguer-14facs-scp}. 

The modules specifying the structure and execution rules
for systems extends the corresponding application modules
with constructors for systems (and embedding systems into
intruder environments) and rules for the extra layer of
message semantics (delivering and collecting). The function

deployApp(sysId,A,idmap) 

takes an application A and a mapping from function blocks 
(identifiers) to devices (their identifiers) and returns a system (with identifier sysId) that is the deployed version
of the application corresponding to the mapping.

Deployment Theorem: 
This theorem states that executions of an application A and a
deployment S = deployApp(sysId,A,idmap) of A are in close
correspondence. In particular the underlying function block
transitions are are the same.


We add an intruder to a system's execution environment in
analogy to the applciation intruder model. It is sufficient
to consider intruders with a finite set of concrete messages
to inject at will as we show that all attacks can be found at
the application level. We extend the deployment
transformation by lifting the badState predicate to systems,
relying on the fact that it depends only on the function
block states.

Deployment Intruder Theorem: 
This theorem states that, letting A,S be as in the Deployment
Theorem, (1) for any execution of [S,msgs] there is a
corresponding execution of [A,emsgs] (where emsgs are the
application level versions of msgs), preserving the
underlying function block transitions; and (2) for any
execution of [A,emsgs] that does not deliver any intruder
messages that should flow on links internal to some device,
has a corresponding execution from [S,msgs].

Corresponding executions preserve the underlying function
block transitions. Thus any attack at the system level has a
corresponding attack at the application level. The condition
in part (2) is because internal messages are protected by the
device execution semantics. If the deployment map is 1-1 then
there are no internal messages, so all [A,emsgs] executions
have a corresponding system level execution.


%%%%%%%%%%%%%%%%%%%%
Wrapping

A device protects communications among its function blocks,
hence the second part of the theorem above. To protect
communications between function blocks on different devices
we use the idea of formal wrapper
\cite{meseguer-etal-08fmoods-cookie} to transform a system S
into a system, wrap(S,emsgs), in which devices are wrapped in
a policy layer protecting communications between devices by
signing messages and checking signatures on flows defined by
emsgs.

Wrapper Theorem
Let A be an application and emsgs a set of message containing
all message sets of size at most $n$ which if injected by an
intruder lead to a badState. The wrapper theorem says that
the wrapped system wrap(deploy(sysId,A,idmap),emsgs) is
resistant to attacks by an $n$ bounded intrudet.


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
\cite{meseguer-etal-08fmoods-cookie}
Rohit Chadha, Carl A. Gunter, Jose ́ Meseguer, Ravinder Shankesi, and Mahesh Viswanathan. Modular preservation of safety properties by cookie-based DoS-protection wrappers. In Proc. FMOODS 2008, volume 5051 of LNCS, pages 39–58. Springer, 2008.

\cite{meseguer-14facs-scp}
Jose ́ Meseguer. Taming distributed system complexity through formal patterns. Sci. Comput. Program., 83:3–34, 2014.