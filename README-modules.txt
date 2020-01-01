Module summary

****** load.maude
basics.maude

  fmod ID 
    sorts Id < Ids  FbC < Cid 

  fmod MYATTRS 
    sorts MyAttr < MyAttrs .

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

emessages.maude
  fmod EMESSAGES is inc PORT . inc EVENTS .
    sort EMsg < EMsgs .
    op {_,_} : Port Event -> EMsg . 
    op size : EMsgs -> Nat .

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

pnp-scenario.maude
  mod PNP-SCENARIO is inc FB-LIB . inc APP-EXE .
    appLinks(id("pnp")) = 
    ops emsgStart emsgI emsgI1 : -> EMsg .
    op pnpInit : EMsg -> Application .
    op pnpInitI : Application EMsgs -> AppIntruder .
    op badState : FBs -> Bool .
    op badState : Application -> Bool .
    op badState : AppIntruder -> Bool .

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

pnp-attacks.maude

  mod PNP-ATTACKS is inc PNP-SCENARIO . inc TRACE2ATTACKS .
    op pnpAttacks : -> EMsgsSet .
    op pnpAttacks1 : -> EMsgsSet .


****** load-deploy
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
    
policy.maude

  fmod POLICY is inc PORT .
    sort iFact < iPolicy .
    sort oFact < oPolicy .
    op [o`:_;_] : Id OutEv -> oFact .  ---  {fbId,out}
    op [i`:_;_,_] : Id InEv Id -> iFact .  --- {fbId,in}, devId

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
    
pnp-scenario-deployed.maude
wrap.maude
pnp-scenario-wrap.maude

