<template>
  <div>
    <!-- This message is thrown by "showMessage" method -->
    <TopMessage :message="message" />
    <fieldset>
      <legend>Input</legend>
      <input type="text" class="bemenet" v-model="bemenet" />
    </fieldset>
    <fieldset>
      <legend>Kezdőállapot</legend>
      <input type="text" class="bemenet xallapot" v-model="kezdoallapot" />
    </fieldset>

    <fieldset style="max-width: 40px">
      <legend>Végállapot</legend>
      <input type="text" class="bemenet xallapot" v-model="vegallapot" />
    </fieldset>

    <fieldset style="max-width: 100px; width: 100px">
      <legend>Maximum iteráció</legend>
      <input type="number" class="bemenet xallapot" v-model="maxIterations" />
    </fieldset>

    <fieldset style="max-width: 140px">
      <legend>Reset state</legend>
      <button style="padding: 12px" @click="reset()">Reset</button>
    </fieldset>

    <fieldset style="max-width: 140px">
      <legend>Clean mode</legend>
      <button style="padding: 12px" @click="hideTrash = !hideTrash">
        {{ hideTrash ? "Show all" : "Clean mode" }}
      </button>
    </fieldset>

    <div class="tablecontainer">
      <table style="table-layout: fixed">
        <thead>
          <tr>
            <th style="width: 50px">Állapotok</th>
            <th v-for="(value, key) in symbolList" :key="key">
              {{ key }}
            </th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(szabaly, szabalyneve) in szabalyok"
            :key="szabalyneve"
            :class="runningRule == szabalyneve ? 'runningrow' : ''"
          >
            <td class="t1">
              <input class="ti1" type="text" :value="szabalyneve" readonly />
            </td>

            <td
              v-for="(alelem, val) in szabaly"
              :key="val"
              :class="
                runningRule == szabalyneve && runningVal == val
                  ? 'runningtd'
                  : ''
              "
            >
              <fieldset
                :class="incorrectState(alelem.uj_allapot) ? 'incorrect' : ''"
              >
                <legend>Új állapot:</legend>
                <input
                  type="text"
                  v-model="alelem.uj_allapot"
                  style="width: 40px"
                />
              </fieldset>

              <fieldset :class="!alelem.uj_ertek && hideTrash ? 'hidden' : ''">
                <legend>Új érték:</legend>
                <input
                  type="text"
                  v-model="alelem.uj_ertek"
                  style="width: 40px"
                />
              </fieldset>

              <fieldset :class="!alelem.irany && hideTrash ? 'hidden' : ''">
                <legend>Irány:</legend>
                <select name="irany" id="" v-model="alelem.irany">
                  <option value="-1">Jobbra</option>
                  <option value="">-</option>
                  <option value="1">Balra</option>
                </select>
              </fieldset>
            </td>
            <td>
              <button class="remove" @click="removeRule(szabalyneve)">
                Töröl
              </button>
            </td>
          </tr>

          <!-- 
                we setup a new row in the table, where we can add new states
            -->
          <tr class="ujszabaly">
            <td>
              <input type="text" v-model="newrulestate" />
            </td>

            <td v-for="(value, key) in symbolList" :key="key">
              <div v-if="newrule[key]">
                <fieldset>
                  <legend>Új állapot:</legend>
                  <input
                    type="text"
                    style="width: 40px"
                    v-model="newrule[key].uj_allapot"
                  />
                </fieldset>

                <fieldset>
                  <legend>Új érték:</legend>
                  <input
                    type="text"
                    style="width: 40px"
                    v-model="newrule[key].uj_ertek"
                  />
                </fieldset>

                <fieldset>
                  <legend>Irány:</legend>
                  <select name="irany" id="" v-model="newrule[key].irany">
                    <option value="-1">Jobbra</option>
                    <option value="" selected>-</option>
                    <option value="1">Balra</option>
                  </select>
                </fieldset>
              </div>
            </td>
            <td>
              <button class="add" @click="addRule()">Hozzáad</button>
            </td>
          </tr>
        </tbody>
      </table>
      <div style="align-self: center">
        <button class="newsymbol" @click="addnewsymbol()">
          Új szimbólum hozzáadása
        </button>
        <input
          type="text"
          class="newsymbol"
          style="border: 2px solid black"
          v-model="newsymbol"
        />
      </div>
    </div>

    <button @click="runTuring()" class="futtat">Futtatás</button>

    <!-- THE TURING STEPS WILL BE RENDERED HERE -->
    <div class="res" v-html="result"></div>
  </div>
</template>
<script>

import defaultRuleset from "../static/defaultRuleset.json";
import TopMessage from "../components/TopMessage"


export default {
  name: "App",
  components: { TopMessage },
  // in this section we define all the instance variables, that can be used anywhere within this app
  data: function () {
    return {
      maxIterations: 30,
      fatalErrorCaught: false,
      runningRule: null,
      runningVal: null,
      newrulestate: null,
      newrule: {},
      newsymbol: "",
      hideTrash: false,

      kezdoallapot: "s",
      vegallapot: "h",
      message: "",

      // this is only to hack to show the running turing steps delayed..
      timingVariable: 0,
      speed: 400, 

      result: "",

      bemenet: "#1110#",
      szabalyok: defaultRuleset,
    };

  },
  computed: {
    
    stateList() {
      //we need to add "vegallapot" manually as it will never be in the ruleset
      return Object.keys(this.szabalyok).concat(this.vegallapot);
    },
    symbolList() {
      return this.szabalyok[Object.keys(this.szabalyok)[0]];
    },
  },
  mounted() {
    if (window.localStorage.getItem("szabalyok") != null) {
      this.szabalyok = JSON.parse(localStorage.getItem("szabalyok"));
    }

    this.alignNewrecord();
  },
  methods: {

    removeRule(szab) {
      delete this.szabalyok[szab];
      this.showMessage("Törölve");
      window.localStorage.setItem("szabalyok", JSON.stringify(this.szabalyok));
    },

    turingProcessing(rules, tape, end, state, i, cell, current) {
      this.fatalErrorCaught = false;
      let iterationCount = 0;
      i = 0;

      // start processing until we do not reach the closing state (by default the 'h' state)
      while (state != end && !this.fatalErrorCaught) {
        iterationCount++;
        if (iterationCount > Number(this.maxIterations)) {
          this.fatalErrorCaught = true;
          this.showMessage("Too many iterations!");
          return false;
        }

        // get the character to be processed
        cell = tape[tape.length - 1 - i];

        try {
          current = rules[state][cell];
        } catch (e) {
          alert(
            "Configuration error: there's no rule specified in the ruleset: [" +
              state +
              "]"
          );
          this.fatalErrorCaught = true;
          return false;
        }
        this.doDebug(tape, i, state, iterationCount);

        // if 'új érték' / value is specified, do the replacement..
        if (current.uj_ertek) tape.splice(i * -1 - 1, 1, current.uj_ertek);

        // if 'irány' / direction is specified, change the 'i' iterator to jump to the correct position (left or right)
        if (current.irany) i += parseInt(current.irany);

        state = current.uj_allapot;
      }

      this.doDebug(tape, i, state, iterationCount);
      
      return tape;
    },

    // this function is to print each steps under each other
    doDebug(tape, i, state, iterationCount) {

      var debug = [];
      tape.forEach((element) => {
        debug.push(element);
      });
      // here we just highlight the affected characted in the chain
      debug.splice(i * -1 - 1, 1, "<b>" + tape[debug.length - i - 1] + "</b>");

      // with this hack we can play with the speed of showing each steps
      setTimeout(() => {
        // the belo two lines are just to tell the template that which Rule and which CELL should be highlighted
        this.runningRule = state;
        this.runningVal = tape[debug.length - i - 1];

        // 'result'  contain the HTML of the printable steps
        this.result +=
          "<span class='nth'>" +
          iterationCount +
          ".</span>" +
          debug.join("") +
          " <i>" +
          state +
          "</i><br>";

        // below we always increase the timingvariable
        // we could avoid this by using ASYNC functions too
      }, (this.timingVariable += this.speed));
    },

    runTuring() {
      this.timingVariable = 0;
      this.result = "";

      // in this step we're calling the turing process
      this.turingProcessing(
        this.szabalyok,
        this.bemenet.split(""),
        this.vegallapot,
        this.kezdoallapot
      );
    },

    reset() {
      window.localStorage.removeItem("szabalyok");
      window.location.reload();
    },

    showMessage(txt) {
      this.message = txt;
      setTimeout(() => {
        this.message = "";
      }, 7000);
    },

    alignNewrecord() {
      this.newrule = JSON.parse(
        JSON.stringify(this.szabalyok[Object.keys(this.szabalyok)[0]])
      );

      // here we basically iterate through each cells of the last row, and clearing out the values - we dont need the first row's values
      Object.keys(this.newrule).forEach((el) => {
        this.newrule[el] = {};
      });
    },

    addnewsymbol() {
      if (this.newsymbol == null || this.newsymbol == "") {
        alert("Blank symbol cannot be added");
        return false;
      }

      if (Object.keys(this.symbolList).indexOf(this.newsymbol) > -1) {
        alert("This symbol has already been added in the list");
        this.newsymbol = "";
        return false;
      }

      Object.keys(this.szabalyok).forEach((element) => {
        this.szabalyok[element][this.newsymbol] = {};
      });
      window.localStorage.setItem("szabalyok", JSON.stringify(this.szabalyok));
      this.szabalyok = JSON.parse(localStorage.getItem("szabalyok"));
      this.showMessage("Hozzáadva");
      this.alignNewrecord();
    },
    addRule() {
      if (!this.newrulestate) {
        alert("Nincs név megadva");
        return;
      }
      this.szabalyok[this.newrulestate] = this.newrule;
      this.showMessage("Hozzáadva");
      this.alignNewrecord();
      window.localStorage.setItem("szabalyok", JSON.stringify(this.szabalyok));
    },

    incorrectState(stat) {
      if (stat == null) return;
      if (this.stateList.indexOf(stat) == -1) {
        this.showMessage(
          "Helytelen konfiguráció. A pirossal jelölt állapotok még nem léteznek."
        );
        return true;
      }
    },
  },

  watch: {
    szabalyok: {
      deep: true,
      handler(val) {
        window.localStorage.setItem("szabalyok", JSON.stringify(val));
      },
    },
  },
};
</script>