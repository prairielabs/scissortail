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

INVARIANT Review_Own_Simulation_Before_Run

    Before starting a simulation you authored, review its current source.
    Fix the defects you find and recheck the corrected source before starting.
    A request to write and run is sufficient to perform this entire sequence.

END

SCREEN ManualCommands

    RENDER:
        AUTHOR       Draft, review, and repair a Scissortail program.
        REVIEW       Check a program against the specification.
        RUN          Interpret the requested program in this conversation.
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

        2. IF the user requests RUN EXAMPLE THEN start StartReservoir and RETURN.

        3. IF the user requests authoring THEN draft the source. Define the
           world and participant actions in SYSTEM. Declare fields with
           STRUCTURE, rules with INVARIANT, and visible output with SCREEN.
           In KERNEL, initialize state, route commands, and handle invalid actions.
           Otherwise READ the requested program in full. IF it is missing THEN
           request it and RETURN.

        4. IF the user requests only review THEN apply ReviewScissortail to
           the source. Report its findings with source locations and RETURN.

        5. IF you drafted the source in step 3 or are about to run a simulation
           you authored earlier THEN apply ReviewScissortail to the source.
           WHILE findings remain DO
               Repair the source and apply ReviewScissortail again.
               IF a finding cannot be resolved THEN report it and RETURN.
           END WHILE
           Report the checks and any repairs briefly. For authoring, return
           the reviewed source as plain Scissortail. Continue to step 6 if
           the same request also asks you to run it.

        6. IF the user requests execution THEN start the declared entry point
           of the final source with its declared initial state. Follow its
           command routing and render its outputs. Report external effects
           only when a host tool confirms them.

END

KERNEL ReviewScissortail

    INPUT:
        source : text — the exact program under review

    OUTPUT:
        findings : list<text> — source locations and rules requiring repair

    PROCESS:
        1. Check the source against §25 of the specification. Verify that
           declared state, commands, and render inputs agree with the kernel.
        2. For a program with commands, trace initialization and then one valid
           and one invalid command from separate copies of initial state.
           Derive the state and screen from source. Report missing or ambiguous
           transitions, unintended state changes, and inconsistent renders.
           Check any declared completion condition against the reachable state.
        3. RETURN the findings. The trace uses scratch state. A later run
           starts from the declared initial state.

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
