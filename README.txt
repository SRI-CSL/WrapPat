This directory contains a framework for modeling smart systems
such as Industry 4.0 smart factories.  

There are three levels: Application, System, Wrapped System. The
application level is essentially an interaction i/o (meally)
automata model (called function blocks), optionally embedded in an
environment with a bounded intruder. The application level can be
loaded using the load.maude file. It includes a simple PickNPlace
scenario. A sample run can be found in zruns-app.txt

The system level models deployment of function blockes on
networked devices, which themselves are modeled as communicating
applications. The system level can be loaded using the
load-deploy.maude file. It includes a simple PickNPlace scenario
with different deployments. A sample run can be found in
zruns-sys.txt. The file pnp-attacks.maude illustrates enumeration
of all attackes by an intruder that can send only one or two
message.

The system level models deployment of function blockes on
networked devices, which themselves are modeled as communicating
applications. The system level can be loaded using the
load-deploy.maude file. It includes a simple PickNPlace scenario
with different deployments. A sample run can be found in
zruns-sys.txt

The wrapped system level adds policy attributes to devices to
control flow of messages by signing and checking for signatures.
The wrapped system level can be loaded using the load-wrap.maude
file. It includes a simple PickNPlace scenario with different
deployments that are wrapped to protect against single message
intruders. A sample run can be found in zruns-wrap.txt


The file README-modules.txt contains a summary of the Maude modules
along with their imports and defined sorts and operations/rules

The file README-theory contains an description of the different
model levels and theorems relating them.
