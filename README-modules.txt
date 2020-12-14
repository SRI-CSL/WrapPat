Module summary

****** load.maude ***************************
***************************
basics.maude

  fmod ID 
    sorts Id < Ids  FbC < Cid 

  fmod MYATTRS 
    sorts MyAttr < MyAttrs .

***************************
fb.maude  requires basics

  fmod EVENTS is inc STRING .
    sorts Event < Events 
    op allEvents : -> Events . 
    op ev : String -> Event .
  
  fmod IO is inc STRING .  
    sorts inEv outEv < IO < IOs
          InEv < Ins < IOs
          OutEv < Outs < IOs
    op inEv : String -> InEv .
    op outEv : String -> OutEv .
    
  fmod PORT is inc ID .  inc IO .
    sorts Port < Ports .
    op {_,_} : Id IO -> Port .
    op {_,_} : Id Port -> Port .
    
    sorts Link < Links .
    op {_,_} : Port Port -> Link .

  fmod EFFECTS is inc IO . inc EVENTS .
    sort InEvEff < InEvEffs .
    sort OutEvEff < OutEvEffs .
    op _:~_ : InEv Event -> InEvEff . 
    op _:~_ : OutEv Event -> OutEvEff . 

  fmod COND is  inc EVENTS . inc IO . inc EFFECTS .
    sort Cond  
    ops tt ff : ->  Cond .
    op _is_ : InEv Event -> Cond [ctor]. 
    op _and_ : Cond Cond -> Cond [assoc comm] .
    op _or_ : Cond Cond -> Cond [assoc comm] .
    op not_ : Cond -> Cond .
    op toNNF : Cond -> Cond .
    op _|=_ : InEvEffs Cond -> Bool .

  fmod TRANSITION is inc COND . 
    sort State < States .
    op st : String -> State [ctor] .
    sort Tr < Trs .
    op tr : State State Cond OutEvEffs -> Tr [ctor] . 
    op filterTrs : Trs State Trs -> Trs .

  fmod FB is inc ID . inc TRANSITION .  inc MYATTRS .
    sort FB < FBs .
    op [_:_|_] : Id FbC MyAttrs -> FB [ctor] . 
    op state`:_ : State -> MyAttr [ctor] .
    op iEvEffs`:_ : InEvEffs -> MyAttr [ctor] .
    op oEvEffs`:_ : OutEvEffs -> MyAttr [ctor] .
    op ticked`:_ : Bool -> MyAttr [ctor] .
    op insFB : FbC -> Ins .
    op outsFB : FbC -> Outs .
    op trsFB : FbC -> Trs .
    op stsFB : FbC -> States .
    op trsFB : FbC State -> Trs .

***************************
emessages.maude
  fmod EMESSAGES is inc PORT . inc EVENTS .
    sort EMsg < EMsgs .
    op {_,_} : Port Event -> EMsg . 
    op size : EMsgs -> Nat .
    op mem : EMsgs EMsg -> Bool .
    op intersect : EMsgs EMsgs -> EMsgs .
    op subset : EMsgs EMsgs -> Bool .
 
    sort EMsgsSet .
    op none : -> EMsgsSet .
    op {_} : EMsgs -> EMsgsSet [ctor] .
    op __ : EMsgsSet EMsgsSet -> EMsgsSet [ctor comm assoc id: none] .
    op size : EMsgsSet -> Nat .
    op addEMsgs : EMsgs EMsgs -> EMsgs .
    op flattenEMsgsSet : EMsgsSet EMsgs -> EMsgs .

  **** lists of emsgs sets, the intent is each list element
  **** has sets of the same size
    sort EMsgssList .
    op nil : -> EMsgssList .
    op [_] : EMsgsSet -> EMsgssList [ctor] .
    op _;_ : EMsgssList EMsgssList -> EMsgssList [ctor assoc id: nil] .
    op len : EMsgssList -> Nat .
    op getNth : EMsgssList Nat -> EMsgsSet .
    op flattenEMsgssList : EMsgssList EMsgs -> EMsgs .


***************************
sym-emessages.maude
  fmod SYM-EMESSAGES is inc EMESSAGES .
    subsort Sym-Event < Event .
    op sev : Nat Nat -> Sym-Event . 
    sort Sym-Id < Id .
    op sid : Nat Nat -> Sym-Id .
    sort Sym-In < InEv .
    op sin : Nat Nat -> Sym-In .
    sort Sym-Out < OutEv .
    op sout : Nat Nat -> Sym-Out .
    sorts IK < IKEle .
    op ik : Event -> IKEle .
    sort SSB < SSBs .
    op _:~_ : Sym-Event Event -> SSB .
    op _:~_ : Sym-Id Id -> SSB .
    op _:~_ : Sym-In InEv -> SSB .
    op _:~_ : Sym-Out OutEv -> SSB .
    op _[_] : Event SSBs -> Event .
    op _[_] : Id SSBs -> Id .
    op _[_] : InEv SSBs -> InEv .
    op _[_] : OutEv SSBs -> OutEv .
    op _[_] : EMsg SSBs -> EMsg .
    op _[[_]] : EMsgs SSBs -> EMsgs .

***************************
sym-transition.maude
  fmod CONSTRAINTS is inc SYM-EMESSAGES . inc COND .
    sort Constraint < CSet .
    op evc : InEv IK -> Constraint .
    sort CSetSet .
    op [_] : CSet -> CSetSet [ctor] .
    op condToCSet : Cond  -> CSetSet .

  fmod SYM-TRANSITION is inc FB . inc CONSTRAINTS .
    subsort SymTr < SymTrs .
    op symtr : State State CSetSet OutEvEffs -> SymTr [ctor] . 
    op tr2symtr : Tr -> SymTr .
    op symtrsFB : Cid State -> SymTrs .
    
  fmod GENSOL is inc SYM-TRANSITION .
    sort SSBsSet .
    op `{_`} : SSBs -> SSBsSet [ctor] .
    op genSol1 : Id EMsgs CSet -> SSBsSet .
    
***************************
app.maude
  fmod APPLICATION is inc FB . inc SYM-EMESSAGES .
    **** GLOBALS set in scenario
    op appFBs : Id -> Ids . **** computable
    op appIns : Id -> Ins .
    op appOuts : Id -> Outs .
    op appLinks : Id -> Links .
  
    sort Application .
    op [_|_] : Id  MyAttrs -> Application .
    op fbs`:_ : FBs -> MyAttr .
    op iEMsgs`:_ : EMsgs -> MyAttr .
    op oEMsgs`:_ : EMsgs -> MyAttr .
    op ssbs`:_ : SSBs -> MyAttr .

    op updateFBs : Application FBs -> Application .
    op addAttr : Application MyAttrs -> Application .
    op setTicked : MyAttrs Bool -> MyAttrs .
    op setTicked : Application Bool -> Application .
    op getTicked :  MyAttrs -> Bool .
    op notApp : MyAttrs -> Bool .

    sort AppIntruder .
    op [_,_] : Application EMsgs -> AppIntruder .
    
  
  fmod APP-AUX is inc APPLICATION . inc EMESSAGES . inc GENSOL .
    op hasSol : FBs EMsgs -> Bool .
    sort EMsgsEMsgs .
    op {_,_} : EMsgs EMsgs -> EMsgsEMsgs [ctor] .
    sort FBsEMsgsEMsgs .
    op {_,_,_} : FBs EMsgs EMsgs -> FBsEMsgsEMsgs [ctor] .
    op extractOutMsgs : Bool FBs FBs EMsgs EMsgs Links 
                      -> FBsEMsgsEMsgs .

  mod APP-EXE is inc APP-AUX .
    crl[app-exe1]
    crl[app-exe2]
    rl[app-intruder]

***************************
fb-lib.maude

  fmod VACUMM-FB is inc FB .
    op vac : -> FbC .
    eq4 insFB(vac) outsFB(vac) stsFB(vac) trsFB(vac)
    op vacInit : Id -> FB .

  fmod TRACK-FB is inc FB .
    op track : -> FbC .
    eq4 insFB(track) outsFB(track) stsFB(track) trsFB(track)
    op trackInit : Id -> FB .

  fmod CONTROL-FB is inc FB .
    op ctl : -> FbC .
    eq4 insFB(ctl) outsFB(ctl) stsFB(ctl) trsFB(ctl)
    op ctlInit : Id -> FB .

  fmod BAD-CONTROL-FB is inc FB .
    op bad-ctl : -> FbC .
    eq4 insFB(bad-ctl) outsFB(bad-ctl) stsFB(bad-ctl) trsFB(bad-ctl)
    op badctlInit : Id -> FB .

  fmod FB-LIB is inc VACUMM-FB . inc TRACK-FB . 
                 inc CONTROL-FB . inc BAD-CONTROL-FB .

  fmod CONTROLX-FB is inc FB .
    op ctlx : -> FbC .
    eq4 insFB(ctlx) outsFB(ctlx) stsFB(ctlx) trsFB(ctlx)
    op ctlxInit : Id -> FB .

fmod FB-LIB-DUAL is inc VACUMM-FB . inc TRACK-FB  .
                    inc CONTROLX-FB . inc PNP-COORD-FB .

***************************
fb-lib1.maude

fmod VACUMM-FB is   inc FB .
*** requires ctl and track input
  op vac : -> FbC .
  op vacInit : Id -> FB .

fmod TRACK-FB is inc FB .
**** orig trac
  op track : -> FbC .
  op trackInit : Id -> FB .

fmod TRACK-FB1 is inc FB .
*** requires ctl and vac input
  op track : -> FbC .
  op trackInit : Id -> FB .

fmod CONTROL-FB is inc FB .
  op ctl : -> FbC .
  op ctlInit : Id -> FB .

fmod FB-LIB is 
  inc VACUMM-FB .  inc TRACK-FB .  inc CONTROL-FB .
fmod FB-LIB1 is
  inc VACUMM-FB . inc TRACK-FB1 .  inc CONTROL-FB .

fmod CONTROLX-FB is inc FB .
**** ctlx -- pauses at each round
  op ctlx : -> FbC .
  op ctlxInit : Id -> FB .

fmod PNP-COORD-FB is  inc FB .
  op pnp-coord : -> FbC .
  op pnp-coordInit : Id -> FB .
 
fmod FB-LIB-DUAL is
  inc VACUMM-FB .  inc TRACK-FB1  .
  inc CONTROLX-FB .  inc PNP-COORD-FB .

***************************
trace2attacks.maude

  mod TRACE2ATTACKS is inc META-LEVEL . inc APP-EXE .
    sort AppIntruder < AppIntList .
    sort EMsgsSet .
    op {_} : EMsgs -> EMsgsSet [ctor] .
    op mtAppInt : -> AppIntruder [ctor] .
    op getBadEMsgs :  Qid AppIntruder -> EMsgsSet .
    *** module, initialConf and search index
      op toAppIntList$ : Module Term Nat -> AppIntList .
      op getEmsgIs : AppIntList EMsgs -> EMsgs .
      op getDelivered : EMsgs SSBs EMsgs -> EMsgs  .

****** load-pnp ***************************
***************************
pnp-scenario.maude   pnp-attacks.maude
  mod PNP-SCENARIO is inc FB-LIB . inc APP-EXE .
    appLinks(id("pnp")) = 
    ops emsgStart emsgI emsgI1 : -> EMsg .
    op pnpInit : EMsg -> Application .
    op pnpInitI : Application EMsgs -> AppIntruder .
    op badState : FBs -> Bool .
    op badState : Application -> Bool .
    op badState : AppIntruder -> Bool .

mod PNP-ATTACKS is inc PNP-SCENARIO . inc TRACE2ATTACKS .
  op pnpAttacks0 : -> EMsgsSet .  --- 0 imsg
  op pnpAttacks : -> EMsgsSet .   --- 1 imsg
  op pnpAttacks1 : -> EMsgsSet .  --- 2 imsgs
  op pnpAttacks2 : -> EMsgsSet .  --- 3 imsgs

****** load-pnp1 ***************************
pnp1-scenario.maude
  mod PNP-SCENARIO is inc FB-LIB1 . inc APP-EXE .
  op pnpInit : EMsg -> Application .
  op pnpInitI : Application EMsgs -> AppIntruder .

  op badState : FBs -> Bool .
  op badState : Application -> Bool .
  op badState : AppIntruder -> Bool .
  op badState1 : FBs -> Bool .
  op badState1 : Application -> Bool .
  op badState1 : AppIntruder -> Bool .

mod PNP-ATTACKS is inc PNP-SCENARIO . inc TRACE2ATTACKS .
using badstate
  op pnpAttacks0 : -> EMsgsSet [memo] . --- no imsgs
  op pnpAttacks1 : -> EMsgsSet [memo] . --- 1 imsg
  op pnpAttacks2 : -> EMsgsSet [memo] . --- 2 imsgs
  op pnpAttacks3 : -> EMsgsSet [memo] . --- 3 imsgs
using badstate1
  op pnp1Attacks0 : -> EMsgsSet [memo] .  --- 0 imsgs
  op pnp1Attacks1 : -> EMsgsSet [memo] .  --- 1 imsg
  op pnp1Attacks2 : -> EMsgsSet [memo] .  --- 2 imsgs
  op pnp1Attacks3 : -> EMsgsSet [memo] .  --- 3 imsgs

****** load-pnp2 ***************************
pnp2-scenario.maude
  mod PNP2-SCENARIO is inc FB-LIB-DUAL . inc APP-EXE .
    ops pnp2Start emsgI emsgI1 : -> EMsg .
    op pnp2FBs : -> FBs .
    op pnp2Init : EMsg -> Application .
    op pnp2InitI : Application EMsgs -> AppIntruder .
    op badState2 : FBs -> Bool .
    op badState2 : Application -> Bool .
    op badState2 : AppIntruder -> Bool .
    op badStateLOnROff : FBs -> Bool .
    op badStateLOnROff : Application -> Bool .
    op badStateLOnROff : AppIntruder -> Bool .

 mod PNP2-ATTACKS is inc PNP2-SCENARIO . inc TRACE2ATTACKS .
    op pnp2Attacks : -> EMsgsSet .
    op pnpLOnROff2Attacks : -> EMsgsSet .
    
****** load-pnp1-2 ***************************
pnp1-2-scenario.maude

**** version with track/vac double checking
mod PNP2-SCENARIO is
  inc FB-LIB-DUAL .
  inc APP-EXE .
  ops pnp2Start emsgI emsgI1 emsgI2 : -> EMsg .
  op pnp2FBs : -> FBs .
  op pnp2Init : EMsg -> Application .
  op pnp2InitI : Application EMsgs -> AppIntruder .
  op badState2 : FBs -> Bool .
  op badState2 : Application -> Bool .
  op badState2 : AppIntruder -> Bool .
  op badState2-1 : FBs -> Bool .
  op badState2-1 : Application -> Bool .
  op badState2-1 : AppIntruder -> Bool .
 
 mod PNP2-ATTACKS is inc PNP2-SCENARIO . inc TRACE2ATTACKS .
   op pnp2Attacks : -> EMsgsSet . --- 1 imsg
   op pnp2Attacks1 : -> EMsgsSet .  --- 2 imsgs
   op pnp2Attacks2 : -> EMsgsSet .  --- 3 imsgs 

***************************
min-attacks.maude
  fmod MIN-ATTACKS is   inc TRACE2ATTACKS .
***** the main goal is to generate minimal protection emsg sets
***** these are minimal sets of emsgs whose protection will prevent attack

**** eml[j] -- the emsg sets of size j+1 from input
  op emsgsSet2emsgsList : EMsgsSet  ->  EMsgssList .

**** return emsgssl st attack sets of size j do not contain any attack set of size  less than j (uses check subsumed)
  op pruneEMsgss : EMsgssList -> EMsgssList .
   
  op checkSubsumed : EMsgs EMsgssList Nat Nat -> Bool .

***** List of pairs [emsg,n]  used to count the occurrences of emsg
***** in a set of attack sets
  sorts EMsgNat EMsgNatL .
  subsort EMsgNat < EMsgNatL .
  op nil : -> EMsgNatL [ctor] .
  op _;_ : EMsgNatL EMsgNatL -> EMsgNatL [ctor assoc id: nil] .
  op [_`,_] : EMsg Nat -> EMsgNat [ctor] .

*** ... [emsg,n] ; ... st emsg occurs in n emsgsets in emsgssl 
*** the first element give max count
  op emssl2emns : EMsgssList  -> EMsgNatL .
  op emssl2emns$ : EMsgssList EMsgs EMsgNatL -> EMsgNatL .
      **** the number of emsg sets with emsg as an element
      op countOccsMSL : EMsgssList EMsg Nat -> Nat .
      op countOccsMS : EMsgsSet EMsg Nat -> Nat .

**** emsgs with max occ count
  op collectMxEMsg : EMsgNatL -> EMsgs .
  op collectMxEMsg$ : EMsgNatL Nat EMsgs -> EMsgs .

***** generate minimal protection emsg sets
**** minimal protection sets for attacks in emsgssl
**** assume emsgssl is pruned, so a minprotset must
**** only intersect each attack set non-trivially
***** genMinProts([none,emsgssl],none)

  sorts EMsgsEML EMsgsEMLs .
  subsort EMsgsEML < EMsgsEMLs .
  op [_,_] : EMsgs EMsgssList -> EMsgsEML .
  op none : -> EMsgsEMLs [ctor] .
  op __ : EMsgsEMLs EMsgsEMLs -> EMsgsEMLs 
         [ctor comm assoc id: none] .

  op genMinProts : EMsgsEMLs EMsgsSet -> EMsgsSet .
  op genMinProtsX : EMsgssList EMsgs EMsgsEMLs EMsgsSet 
                  -> EMsgsSet .
*****              cur    cxt   todo     result

***** Given maxocc set,and emsgssl produce
***** [m,removeM(emsgssl)] : m in maxocc
***** given [emsgs,emsgssl] refine with [emsgs m, emsgssl ]
*****    [m,removeM(emsgsl,m)] : m in maxocc(emsgsl)
   op emsgssl2oneprot : EMsgssList -> EMsgsEMLs .
      op dropHasEmsg : EMsgssList EMsg EMsgssList -> EMsgssList .
      op removeHasEmsg : EMsgsSet EMsg EMsgsSet -> EMsgsSet .

  ***** { {emsgs emsg} | [emsg,nil] in emsgsemls1}
   op getMPs : EMsgsEMLs EMsgs EMsgsSet -> EMsgsSet .
  **** { [emsgs emsg, emsgssl] st [emsg, emsgssl] in emsgss1 
  ****   and emsgssl =/= nil
   op getTodos :  EMsgsEMLs EMsgs EMsgsEMLs -> EMsgsEMLs .

******************************************************
****** load-deploy  ***************************
messages.maude

  fmod MESSAGES is inc PORT . inc EVENTS .
    sort PMsg < Msg < Msgs .
    op {_,_,_} : Port Port Event -> PMsg . 
    op size : Msgs -> Nat .

  fmod SIGNED-MESSAGES is inc MESSAGES .
    sort SignedMsg < Msg .
    op sg : Event Id -> Event .
    op signMsg : PMsg Id -> Msg .
    op unsign : Event -> Event .
    op isSigned : Event Id -> Bool .
    
***************************
policy.maude

  fmod POLICY is inc PORT .
    sort iFact < iPolicy .
    sort oFact < oPolicy .
    op [o`:_;_] : Id OutEv -> oFact .  ---  {fbId,out}
    op [i`:_;_,_] : Id InEv Id -> iFact .  --- {fbId,in}, devId

***************************
sys.maude

  fmod SYSTEM is inc APPLICATION . inc MESSAGES . inc MAP{Id,Id} .
                 inc POLICY . ---- wrapped
    **** GLOBALS set in scenario
    op sysIns : Id -> Ins .
    op sysOuts : Id -> Outs .
    op sysMap : Id -> Map{Id,Id} .
    sorts System SysIntruder .
    op [_|_|_] : Id Id MyAttrs -> System .
    op devs`:_ : Apps -> MyAttr .
    op iMsgs`:_ : Msgs -> MyAttr .
    op oMsgs`:_ : Msgs -> MyAttr .
    op iPol`:_ : iPolicy -> MyAttr .   ---- wrapped
    op oPol`:_ : oPolicy -> MyAttr .   ---- wrapped
    op [_,_] : System Msgs -> SysIntruder [ctor] .
    op mkSys : Id Id Apps Msgs -> System .
    op addAttr : System MyAttrs -> System .

  fmod SYS-AUX is inc SYSTEM . inc APP-AUX . 
                  inc SIGNED-MESSAGES .  ---- wrapped
    op isDone : Apps -> Bool .
    op deliver2Devs : Apps Msgs Links Map{Id,Id} -> Apps .
    op checkMsg : Id Port Event Links Map{Id,Id} MyAttrs -> EMsgs .
    op checkPolicy : Port Event Links Map{Id,Id} MyAttrs -> EMsgs .
    op checkPort : Id EMsgs Links Map{Id,Id} -> EMsgs .
    op resetTicks : Apps Apps -> Apps .
    sort DevsMsgsMsgs .
    op {_,_,_} : Apps Msgs Msgs -> DevsMsgsMsgs [ctor] .
    op extractOutMsgs : Id Apps Apps Msgs Msgs 
                        Links Map{Id,Id}  -> DevsMsgsMsgs .
    op emsgs2omsgs : Id Id EMsgs Msgs Links -> Msgs .

  mod SYS-EXE is inc SYS-AUX . inc APP-EXE .
    crl[sys-deliver]
    crl[sys-collect]
    rl[sys-intruder]
    
***************************
pnp-scenario-deployed.maude

  fmod DEPLOY-APP is inc APPLICATION . inc SYSTEM .
    op deployFBs : FBs Apps Map{Id,Id} -> Apps .
    op emsg2imsg : Id EMsg Map{Id,Id}  -> Msgs .  --- 0 or 1

  mod PNP-SCENARIO-DEPLOYED is inc SYS-EXE . inc PNP-SCENARIO .
                               inc DEPLOY-APP .
    ops msgStart msgI msgI1 : -> Msg .
    ops Dev1 Dev2 Dev3 : -> Application .
    op pnpD1-1Init : Msgs -> System .
    op pnpDInitI : System Msgs -> SysIntruder .
    op badState : System -> Bool .
    op badState : SysIntruder -> Bool .
    ops dPnP1-1 dPnP-vc-t : -> System .

****** load-wrap  **************************
wrap.maude

  fmod WRAP-SYSTEM is inc APPLICATION . inc SYSTEM . inc POLICY .
                      inc SIGNED-MESSAGES .
    op addEMsgs : EMsgs EMsgs -> EMsgs .
    op flattenEMsgsSet : EMsgsSet EMsgs -> EMsgs .
    op wrap-sys : System EMsgsSet -> System .
    op wrap-devs : Apps EMsgs Links Map{Id,Id} Apps -> Apps . 
    op wrap-dev : Application EMsgs Links Map{Id,Id} 
                 iPolicy oPolicy -> Application . 

***************************
pnp-scenario-wrap.maude

  mod PNP-SCENARIO-WRAP is inc PNP-SCENARIO-DEPLOYED . 
                 inc TRACE2ATTACKS . inc WRAP-SYSTEM .
    op pnpBad : -> EMsgsSet .
    ops wPnP1-1 wPnP-vc-t : -> System .

