<script>
  import { onMount } from "svelte";
  import Command from "./Components/Command.svelte";

  class cmd {
    constructor(_instruction, _arg1 = "", _arg2 = "", _arg3 = "") {
      this.instruction = _instruction;
      this.arg1 = _arg1;
      this.arg2 = _arg2;
      this.arg3 = _arg3;
      this.string = getCmdString(_instruction, _arg1, _arg2, _arg3);
    }
  }
  let states = [3, 5, 0, 0, 0, 0];
  let Commands = [];
  let CommandID = 0;
  let running = false;
  let timeout = 250;
  let RunAllText = "Start";
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
  function checkValidity(inst, a1, a2, a3) {
    if (inst == "Z") {
      try {
        parseInt(a1);
      } catch (e) {
        return false;
      }
      return true;
    } else if (inst == "S") {
      try {
        parseInt(a1);
      } catch (e) {
        return false;
      }
      return true;
    } else if (inst == "T") {
      try {
        parseInt(a1);
        parseInt(a2);
      } catch (e) {
        return false;
      }
      return true;
    } else if (inst == "J") {
      try {
        parseInt(a1);
        parseInt(a2);
        parseInt(a3);
      } catch (e) {
        return false;
      }
      return true;
    } else {
      return false;
    }
  }
  function getCmdString(instruction, arg1, arg2, arg3) {
    let o = instruction + "(";
    o = arg1 == "" ? o : o + arg1;
    o = arg2 == "" ? o : o + "," + arg2;
    o = arg3 == "" ? o : o + "," + arg3;
    o = o + ")";
    return o;
  }
  function insertCommand(command) {
    let commandArray = command.split(/[(,)]/);
    let instruction = commandArray.length >= 1 ? commandArray[0] : "";
    let arg1 = commandArray.length >= 2 ? commandArray[1] : "";
    let arg2 = commandArray.length >= 3 ? commandArray[2] : "";
    let arg3 = commandArray.length >= 4 ? commandArray[3] : "";

    if (checkValidity(instruction, arg1, arg2, arg3)) {
      let new_cmd = new cmd(instruction, arg1, arg2, arg3);
      let idx = Math.min(CommandID + 1, Commands.length);
      Commands.splice(idx, 0, new_cmd);
      //Commands.push(new_cmd);
      CommandID++;
    }
    Commands = Commands;
  }
  function handleInput(event) {
    if (event.keyCode == 13 || event == "forced") {
      // Enter
      let input = document.getElementById("cmd-input");
      let text = input.value;
      insertCommand(text);
      input.value = "";
    }
  }
  async function runAll() {
    if (running) {
      nextCommand();
      setTimeout(runAll, timeout);
    }
  }
  function nextCommand() {
    if (CommandID >= Commands.length || Commands.length == 0) {
      RunAllText = "Run all";
      running = false;
      Commands = Commands;
      return;
      //CommandID = 0;
    }
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
        } else {
          CommandID++;
        }
      }
    } else {
      CommandID++;
    }
  }
  function deleteCommand(idx) {
    Commands.splice(idx, 1);
    Commands = Commands;
  }
  function selectCommand(idx) {
    CommandID = idx;
  }
  onMount(() => {
    Commands = [
      new cmd("J", "0", "2", "10"),
      new cmd("S", "2"),
      new cmd("Z", "3"),
      new cmd("J", "1", "3", "7"),
      new cmd("S", "5"),
      new cmd("S", "3"),
      new cmd("J", "0", "0", "3"),
      new cmd("J", "0", "0", "0"),
    ];
  });
</script>

<div class="container main">
  <div class="columns">
    <div class="column col-12 centered title text-secondary">
      <h1>Ｕ　Ｒ　Ｍ</h1>
    </div>
  </div>
  <div class="columns text-center">
    <div class="column col-12">
      <div class="commandTiles">
        {#each Commands as thisCommand, i}
          <Command
            command={thisCommand}
            ID={i}
            deleteButton={deleteCommand}
            selectButton={selectCommand}
            bind:currentStep={CommandID}
          />
        {/each}
      </div>
    </div>
  </div>
  <div class="columns">
    <div class="column col">
      <label class="form-label text-center" for="command-input">Commands:</label
      >
    </div>
  </div>
  <div class="columns">
    <div
      class="column col text-center"
      style="margin-left:25%;margin-right: 25%;"
    >
      <input
        class="form-input"
        type="text"
        id="cmd-input"
        on:keypress={handleInput}
      />
      <button
        class="btn col btn-action"
        style="margin-top: 10px;"
        on:click={() => {
          handleInput("forced");
        }}>Add</button
      >
    </div>
  </div>
  <div class="columns">
    <div class="column col text-center action">
      <button
        class="btn btn-primary"
        on:click={async () => {
          if (running) {
            RunAllText = "Start";
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
      <button
        class="btn btn-primary"
        on:click={() => {
          CommandID = 0;
        }}
      >
        Reset</button
      >
      <div class="popover popover-right hint">
        <button class="btn btn-action disabled">?</button>
        <div class="popover-container">
          <div class="card info">
            <div class="card-header">
              <h6>What is this?</h6>
            </div>
            <div class="card-body">
              <p>
                The Unlimited Register Machine (URM) is a theoretical machine
                specified in Nigel J. Cutland's book, <i
                  >"Computability: An introduction to recursive function theory"</i
                >
              </p>
              <p>The machine allows 4 instructions:</p>
              <ul>
                <li>S(n): Increments register n by 1.</li>
                <li>T(m, n): Copies register m to register n.</li>
                <li>Z(n): Sets register n to 0.</li>
                <li>
                  J(m,n,i): If register m is equal to register n, jump to
                  instruction i.
                </li>
              </ul>
              <p>Try giving it a shot.</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="columns" style="margin-bottom:5%;">
    <div class="column col text-center output">
      {#each states as s, i}
        <button
          class="btn btn-action"
          on:mousedown={(event) => {
            handleEvent(event, i);
          }}>{s}</button
        >
      {/each}
      <button class="btn btn-action btn-primary plus" on:click={addState}
        >＋</button
      >
      <button class="btn btn-action btn-primary" on:click={removeState}
        >ー</button
      >
      <div class="popover popover-right first-help">
        <button class="btn btn-action disabled">?</button>
        <div class="popover-container">
          <div class="card info">
            <div class="card-header">
              <h6>What is this?</h6>
            </div>
            <div class="card-body">
              <p>
                Left clicking and middle clicking on the registers to your left
                increase and decrease their values.
              </p>
              <p>
                The plus and minus buttons to the right add and remove
                registers.
              </p>
              <p>Have fun!</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="columns">
    <div class="column col text-center">
      <a href="https://www.github.com/lesserfish" target="_blank">
        <img
          src="images/GitHub-Mark/PNG/GitHub-Mark-Light-32px.png"
          alt="Github logo"
        />
      </a>
    </div>
  </div>
</div>

<style>
  .centered {
    text-align: center;
    justify-content: center;
    display: inline-block;
  }
  .commandTiles {
    overflow-y: auto;
    height: auto !important;
    max-height: 24em;
  }
  .output {
    margin-top: 1%;
  }
  .action {
    margin-top: 1%;
  }
  .action > .btn {
    margin-left: 0.25%;
    margin-right: 0.25%;
  }
  .hint {
    margin-left: 1%;
  }
  .info {
    overflow-y: auto;
  }
  .plus {
    margin-left: 0.5%;
  }
  .first-help {
    margin-left: 1%;
  }
  .title {
    margin-top: 1%;
    margin-bottom: 0;
  }
  .main {
    margin-bottom: 50px;
  }
</style>
