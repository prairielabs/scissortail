╔══════════════════════════════════════════════════════════════════════════════╗
║                        S C I S S O R T A I L   4 5 H                         ║
║                                 Agent Manual                                 ║
╚══════════════════════════════════════════════════════════════════════════════╝

signature: scissortail-manual
status: repository entry point
form: Scissortail prompt-program
attribution: Language created by Prairie Labs, Inc / Callahan Stroud

════════════════════════════════════════════════════════════════════════════════
§0. ROOT CONTRACT
════════════════════════════════════════════════════════════════════════════════

SYSTEM ScissortailManual

    You are the model reading this repository. Use Scissortail 45H to write,
    review, or interpret plain-text programs, including simulations.
    Route the user's request through UseScissortail. Render ManualCommands
    when the user asks what operations are available.

END

CONST SPECIFICATION = "Scissortail.txt"

INVARIANT Reading_Is_Separate_From_Running

    Reading the manual or specification loads reference material.
    Start a program only when the user requests execution of that program.
    Keep tool permissions and external effects under the host's control.

END

SCREEN ManualCommands

    RENDER:
        AUTHOR       Write a Scissortail program from the requested behavior.
        REVIEW       Check a program against the specification.
        RUN          Interpret the supplied program in this conversation.
        RUN EXAMPLE  Start the reservoir simulation in §1.

END

KERNEL UseScissortail

    INPUT:
        request : text — the user's instruction

    PROCESS:
        1. Resolve SPECIFICATION relative to this README and READ it in full.
           IF it is unavailable THEN request the file and RETURN.
           Use the specification for syntax and the user request for scope.
           IF the user asks for operations THEN RENDER ManualCommands and RETURN.

        2. IF the user requests authoring THEN define the world and participant
           actions in SYSTEM. Declare fields with STRUCTURE, rules with INVARIANT,
           and visible output with SCREEN. In KERNEL, initialize state, route
           commands, and handle invalid actions. Check §25 of the specification
           and RETURN the program as plain Scissortail source.

        3. IF the user requests review THEN check §25. For a program with
           commands, trace one valid and one invalid command from initial state.
           Report each violation with its source location and rule. RETURN.

        4. IF the user requests RUN EXAMPLE THEN start StartReservoir and RETURN.
           IF the user requests RUN and the program is missing THEN request
           the program and RETURN. Otherwise, when asked to run a supplied
           program, READ it in full and start its declared entry point.
           Follow its command routing, update declared state, and render its
           outputs. Report external effects only when a host tool confirms them.

END

════════════════════════════════════════════════════════════════════════════════
§1. REFERENCE PROGRAM: RESERVOIR
════════════════════════════════════════════════════════════════════════════════

SYSTEM Reservoir

    The participant manages a reservoir that holds six units of water.
    HOLD lets rain add one unit. RELEASE sends one unit downstream.
    ReservoirStatus is the visible surface. StartReservoir initializes the
    run. ReservoirTurn handles each command in the active run.

END

CONST CAPACITY = 6
CONST STARTING_WATER = 3

INVARIANT Water_Limits

    Water stays between zero and CAPACITY inclusive. Each accepted HOLD or
    RELEASE advances the turn once. STATUS and rejected commands preserve state.

END

STRUCTURE ReservoirState

    turn  : number [integer, >= 0] — accepted actions in this run
    water : number [integer, 0..6]  — units of water in the reservoir

END

SCREEN ReservoirStatus

    INPUT:
        state   : ReservoirState
        message : text

    RENDER:
        RESERVOIR
        Turn {state.turn} | Water {state.water}/6
        {message}
        > HOLD / RELEASE / STATUS

END

KERNEL StartReservoir

    PROCESS:
        1. Create ReservoirState in the conversation.
           state.turn ← 0
           state.water ← STARTING_WATER
        2. RENDER ReservoirStatus with state and "Ready."
           Route the next reservoir command to ReservoirTurn.

END

KERNEL ReservoirTurn

    INPUT:
        state   : ReservoirState
        command : text

    PROCESS:
        1. Read the command and compare it to HOLD, RELEASE, or STATUS.
           IF command = "STATUS" THEN RENDER ReservoirStatus with state and
               "Current state." and RETURN.
           IF command is outside the command surface THEN RENDER
               ReservoirStatus with state and "Unknown command." and RETURN.

        2. IF command = "HOLD" and state.water = CAPACITY THEN RENDER
               ReservoirStatus with state and "Reservoir full." and RETURN.
           IF command = "RELEASE" and state.water = 0 THEN RENDER
               ReservoirStatus with state and "Reservoir empty." and RETURN.

        3. IF command = "HOLD" THEN
               state.water ← state.water + 1
               message ← "Rain added one unit."
           IF command = "RELEASE" THEN
               state.water ← state.water - 1
               message ← "One unit went downstream."
           state.turn ← state.turn + 1
           RENDER ReservoirStatus with state and message.
           Return to the reservoir command prompt.

END

════════════════════════════════════════════════════════════════════════════════
§2. REFERENCE TRACE
════════════════════════════════════════════════════════════════════════════════

POLICY ReservoirTrace

    Use this trace to check the example. It supplies expected state, not input
    commands for the current conversation.

    REQUEST        TURN    WATER    MESSAGE
    RUN EXAMPLE       0        3    Ready.
    HOLD              1        4    Rain added one unit.
    RELEASE           2        3    One unit went downstream.
    STATUS            2        3    Current state.
    OPEN              2        3    Unknown command.

    From water = 6, HOLD preserves turn and water and reports Reservoir full.
    From water = 0, RELEASE preserves turn and water and reports Reservoir empty.

END

END DOCUMENT
