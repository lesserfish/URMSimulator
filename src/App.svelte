<script>
import { onMount } from "svelte";


  class cmd {
    constructor(original, _instruction, _arg1 = "", _arg2 = "", _arg3 = "") {
      this.instruction = _instruction;
      this.arg1 = _arg1;
      this.arg2 = _arg2;
      this.arg3 = _arg3;
      this.string = original;
    }
  }
  let states = [3, 5, 0, 0, 0, 0];
  let CommandForm = "J(0,2,10)\nS(2)\nZ(3)\nJ(1, 3, 7)\nS(5)\nS(3)\nJ(0,0,3)\nJ(0,0,0)";
  let Commands = [];
  let CommandText = "";
  let CommandID = 0;
  let running = false;
  let timeout = 250;
  let RunAllText = "Run All";
  let modalState = "";
  function increaseValue(i) {
    states[i] = states[i] + 1;
    states = states;
  }
  function decreaseValue(i) {
    states[i] = states[i] > 0 ? states[i] - 1 : 0;
    states = states;
  }
  function handleEvent(event, i) {
    if (event.which == 1) {
      increaseValue(i);
    } else if (event.which == 2) {
      decreaseValue(i);
    }
  }
  function addState() {
    states.push(0);
    states = states;
  }
  function removeState() {
    states.pop();
    states = states;
  }
  function updateCommands() {
    Commands = [];
    let cmd_array = CommandForm.split("\n");
    for (let i = 0; i < cmd_array.length; i++) {
      let _cmd = cmd_array[i];
      let sections = _cmd.split(/[(,)]/);
      let instruction = sections.length >= 1 ? sections[0] : "";
      let arg1 = sections.length >= 2 ? sections[1] : "";
      let arg2 = sections.length >= 3 ? sections[2] : "";
      let arg3 = sections.length >= 4 ? sections[3] : "";

      let new_cmd = new cmd(_cmd, instruction, arg1, arg2, arg3);
      Commands.push(new_cmd);
    }
    Commands = Commands;
    CommandID = CommandID;
    CommandText = Commands[CommandID].string;
  }
  async function runAll() {
    if (running) {
      nextCommand();
      setTimeout(runAll, timeout);
      console.log("Loop");
    }
  }
  function nextCommand() {
    //	try{
    let CurrentCommand = Commands[CommandID];
    if (CurrentCommand.instruction == "Z") {
      let idx = parseInt(CurrentCommand.arg1);
      if (states.length > idx) {
        states[idx] = 0;
      }
      states = states;
      CommandID++;
    } else if (CurrentCommand.instruction == "S") {
      let idx = parseInt(CurrentCommand.arg1);
      if (states.length > idx) {
        states[idx] += 1;
        states = states;
      }
      CommandID++;
    } else if (CurrentCommand.instruction == "T") {
      let idx = parseInt(CurrentCommand.arg1);
      let idy = parseInt(CurrentCommand.arg2);
      if (states.length > idy && states.length > idx) {
        states[idy] = states[idx];
        states = states;
      }
      CommandID++;
    } else if (CurrentCommand.instruction == "J") {
      let idx = parseInt(CurrentCommand.arg1);
      let idy = parseInt(CurrentCommand.arg2);
      let idz = parseInt(CurrentCommand.arg3);
      if (states.length > idx && states.length > idy) {
        if (states[idy] == states[idx]) {
          CommandID = idz;
          states = states;
        }
        else
        {
          CommandID++;
        }
      }
    } else {
      CommandID++;
    }

    if (CommandID >= Commands.length) {
      CommandID = 0;
    }
    CommandText = Commands[CommandID].string;

  }

  onMount( () => {
    updateCommands();
  });
</script>

<div class="container">
  <div class="columns">
    <div class="column col-12 centered title text-secondary"><h1>Ｕ　Ｒ　Ｍ</h1></div>
  </div>
  <div class="columns">
    <div class="column col">
      <div class="form-group col commands">
        <label class="form-label text-center" for="command-input"
          >Commands:</label
        >
        <textarea
          class="form-input"
          id="command-input"
          placeholder=""
          rows="8"
          bind:value={CommandForm}
          on:change={updateCommands}
        />
      </div>
    </div>
  </div>
  <div class="columns">
    <div class="column col text-center action">
      <button
        class="btn btn-primary"
        on:click={async () => {
          if (running) {
            RunAllText = "Run all";
            running = false;
          } else {
            RunAllText = "Stop";
            running = true;
            runAll();
          }
        }}
      >
        {RunAllText}</button
      >
      <button class="btn btn-primary" on:click={nextCommand}> Next</button>
      <button class="btn btn-primary" on:click={() => {(CommandID = 0); CommandText = Commands[CommandID].string;}}> Reset</button>  
      <div class="popover popover-right hint">
        <button class="btn btn-action disabled">?</button>
        <div class="popover-container">
          <div class="card info">
            <div class="card-header">
              <h6>What is this?</h6>
            </div>
            <div class="card-body">
                <p>The Unlimited Register Machine (URM) is a theoretical machine specified in Nigel J. Cutland's book, <i>"Computability: An introduction to recursive function theory"</i></p>
                <p>The machine allows 4 instructions: </p>
                <ul>
                  <li>S(n): Increments register n by 1.</li>
                  <li>T(m, n): Copies register m to register n.</li>
                  <li>Z(n): Sets register n to 0.</li>
                  <li>J(m,n,i): If register m is equal to register n, jump to instruction i.</li>
                </ul>
                <p>Try giving it a shot.</p>
              </div>
          </div>
        </div>
      </div>

    </div>
  </div>
  <div class="columns">
    <div class="column col text-center console">
      <p>Current command({CommandID}): {CommandText}</p>
    </div>
  </div>
  <div class="columns">
    <div class="column col text-center output">
      {#each states as s, i}
        <button
          class="btn btn-action"
          on:mousedown={(event) => {
            handleEvent(event, i);
          }}>{s}</button
        >
      {/each}
      <button class="btn btn-action btn-primary plus" on:click={addState}>＋</button>
      <button class="btn btn-action btn-primary" on:click={removeState}>ー</button>
      <div class="popover popover-right first-help">
        <button class="btn btn-action disabled">?</button>
        <div class="popover-container">
          <div class="card info">
            <div class="card-header">
              <h6>What is this?</h6>
            </div>
            <div class="card-body">
              <p> Left clicking and middle clicking on the registers to your left increase and decrease their values.</p> 
              <p>The plus and minus buttons to the right add and remove registers. </p>
              <p>Have fun!</p>
              </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</div>

<style>
  .centered {
    text-align: center;
    justify-content: center;
    display: inline-block;
  }
  .commands {
    margin-left: 30%;
    margin-right: 30%;
  }
  .output {
    margin-top: 1%;
  }
  .action {
    margin-top: 5%;
  }
  .action > .btn {
    margin-left: 0.25%;
    margin-right: 0.25%;
  }
  .console {
    margin-top: 1%;
  }
  .hint{
    margin-left: 1%;
  }
  .info{
    overflow-y: auto;
  }
  .plus{
    margin-left:0.5%;
  }
  .first-help{
    margin-left:1%;
  }
  .title{
    margin-top:1%;
    margin-bottom: 0;
  }

</style>
