This directory contains a framework for modeling smart
systems such as Industry 4.0 smart factories.

There are three levels: Application, System, Wrapped System.
The application level is essentially an interaction i/o
(meally) automata model (called function blocks), optionally
embedded in an environment with a bounded intruder. The
application level can be loaded using the load.maude file.
It includes a simple PickNPlace scenario. A sample run can
be found in zruns-app.txt [load1.maude loads a version with
vac-track coordinating to avoid some attacks.]

The system level models deployment of function blockes on
networked devices, which themselves are modeled as
communicating applications. The system level can be loaded
using the load-deploy.maude file. It includes a simple
PickNPlace scenario with different deployments. A sample run
can be found in zruns-sys.txt. The file pnp-attacks.maude
illustrates enumeration of all attackes by an intruder that
can send only one or two message.

The system level models deployment of function blockes on
networked devices, which themselves are modeled as
communicating applications. The system level can be loaded
using the load-deploy.maude file. It includes a simple
PickNPlace scenario with different deployments. A sample run
can be found in zruns-sys.txt

The wrapped system level adds policy attributes to devices
to control flow of messages by signing and checking for
signatures. The wrapped system level can be loaded using the
load-wrap.maude file. It includes a simple PickNPlace
scenario with different deployments that are wrapped to
protect against single message intruders. A sample run can
be found in zruns-wrap.txt

A more complex scenario, PnP2 can be loaded using
load-pnp2.maude. This scenario has two copies of the PnP
application synchronized by a coord function block. A sample
run can be found in zruns-pnp2-app.txt and the successful
intruder attackes are computed at the end of
pnp2-scenario.maude

In summary:
pnp  --  is the orginal model
pnp1 --  is pnp modified so vac-track coordinate
pnp2 --  has two pnp's with coordinator controler
pnp1-2 - is pnp1 built with pnp1s

You can search for attacks by starting Maude with one of the following:
load-pnp.maude  -- loads pnp scenario and attacks
load-pnp1.maude -- loads pnp1 scenario and attacks 
load-pnp2.maude -- loads pnp2 scenario pnp2-scenario 
load-pnp1-2.maude  loads pnp1-2 scenario pnp1-2-scenario 

Each scenario/attack file contains sample commands including
enumerating all attacks for some bounded intruders.
  
min-attack-pnp<x> files load the corresponding scenario
and define computations for the minimal protection sets for the
enumerated attack sets.  

The file README-modules.txt contains a summary of the Maude
modules along with their imports and defined sorts and
operations/rules

The file README-theory contains an description of the
different model levels and theorems relating them.
