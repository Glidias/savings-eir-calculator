<template>
  <div id="app">
  <h1 style="text-align:center"><i>o899u</i></h1>
  <h2> 
  Savings Account EIR Calculator
  </h2>
  <p>
   Your balance: $<input type="number" v-model.number="balance">
  </p>
  Denominations:
  <table border="1">
    <thead>
      <tr>
        <td>Amount</td>
        <td>Interest</td>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(d,i) in denominations" :key="i">
         <td>$<input type="number" min="0" :disabled="1 && denominations.length > 1 && i+1===denominations.length" :value="i + 1 === denominations.length ? lastAmt : d.amt" @change="amtChange(i, $event.target.value)" @input="amtChange(i, $event.target.value)"></td>
         <td><input type="number" step="0.01" min="0" :value="d.itr" @input="itrChange(i, $event.target.value )" @change="itrChange(i, $event.target.value )" >%</td>
      </tr>
    </tbody>
  </table>
  <button @click="add()" :disabled="false">+</button>
  <button @click="minus()" :disabled="denominations.length===1">-</button>
  <p>
   Total Annual Interest Earned: ${{totalEarned.toFixed(2)}}<br>
   (Per Month ~): ${{(totalEarned/12).toFixed(2)}}
  </p>
  <p>
    EIR: {{eir.toFixed(3)}}%
  </p>
  <div style="float:right; margin-bottom:15px ">
     name: <input class="filenamebox" type="text" v-model="saveFileName">
    <br><button :disabled="!saveFileName" @click="onAddCatalogClick()">{{isEditing ? 'Save' : 'Add to Catalog'}}</button>
    <span class="filenamebox" v-if="isEditing">{{catalog[editIndex].name}}</span>
    <br><button style="float:right" @click="deleteItemAt(editIndex)" v-show="isEditing">Delete</button>
  </div>
  <hr style="clear:both">
  <p>
    <b>Catalog List (click to reset):</b>
    <ul>
      <li v-for="(c,i) in catalog" :key="i">
          <a @click="activateCatalogItem(c)">{{c.name}}</a>
          <span v-show="c.editable">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a @click="editItemAt(i)">✎</a></span>
       </li>
      </ul>
    </p>
  <hr>
  <p>Source code: <a :href="SRC_LINK" target="_blank">Link</a></p>
</div>
</template>

<script>
const SRC_LINK = "https://codesandbox.io/s/savings-o899u?file=/src/App.vue";
export default {
  name: "App",
  data: function () {
    var catalog = [
       {
        name: "UOB One ($500ccSpd + credit sal >=)(>=May 2024)",
        editable: false,
        denom: [
          { amt: 75000, itr: 3 },
          { amt: 50000, itr: 4.5 },
          { amt: 25000, itr: 6 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "UOB Stash (>=Mar 2023)",
        editable: false,
        denom: [
          { amt: 10000, itr: 0.05 },
          { amt: 30000, itr: 2 },
          { amt: 30000, itr: 3 },
          { amt: 30000, itr: 5 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "UOB One ($500ccSpd + credit sal >=)(>=Dec 2022)",
        editable: false,
        denom: [
          { amt: 30000, itr: 3.85 },
          { amt: 30000, itr: 3.9 },
          { amt: 15000, itr: 4.85 },
          { amt: 25000, itr: 7.8 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "UOB One ($500ccSpd + credit sal >=)(>=Sep 2022)",
        editable: false,
        denom: [
          { amt: 15000, itr: 1.4 },
          { amt: 15000, itr: 1.4 },
          { amt: 15000, itr: 1.5 },
          { amt: 15000, itr: 1.5 },
          { amt: 15000, itr: 2.5 },
          { amt: 25000, itr: 3.6 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "CIMB FastSaver Only (>=Nov 2022)",
        editable: false,
        denom: [
          { amt: 10000, itr: 1.5 },
          { amt: 15000, itr: 1.5 },
          { amt: 25000, itr: 2.5 },
          { amt: 25000, itr: 3.5 },
          { amt: 0, itr: 0.8 },
        ],
      },
      {
        name: "UOB One (max tier)(>=Nov 2020)",
        editable: false,
        denom: [
          { amt: 15000, itr: 0.5 },
          { amt: 15000, itr: 0.55 },
          { amt: 15000, itr: 0.65 },
          { amt: 15000, itr: 0.8 },
          { amt: 15000, itr: 2.5 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "UOB One (max tier)(>=Aug 2020)",
        editable: false,
        denom: [
          { amt: 15000, itr: 0.75 },
          { amt: 15000, itr: 0.85 },
          { amt: 15000, itr: 0.9 },
          { amt: 15000, itr: 1.0 },
          { amt: 15000, itr: 2.5 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "UOB One (max tier) < Aug 2020",
        editable: false,
        denom: [
          { amt: 15000, itr: 1.25 },
          { amt: 15000, itr: 1.3 },
          { amt: 15000, itr: 1.35 },
          { amt: 15000, itr: 1.4 },
          { amt: 15000, itr: 3.68 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "UOB Stash (>=Nov 2020)",
        editable: false,
        denom: [
          { amt: 10000, itr: 0.05 },
          { amt: 30000, itr: 0.3 },
          { amt: 30000, itr: 0.6 },
          { amt: 30000, itr: 1.0 },
          { amt: 0, itr: 0.05 },
        ],
      },
      {
        name: "UOB Stash < Nov 2020",
        editable: false,
        denom: [
          { amt: 10000, itr: 0.05 },
          { amt: 40000, itr: 0.8 },
          { amt: 50000, itr: 1 },
          { amt: 0, itr: 0.05 },
        ],
      },
    ];
    const initCatalogLen = catalog.length;
    //localStorage.clear();
    var customCatalog = localStorage.getItem("customCatalog");
    if (customCatalog) {
      customCatalog = JSON.parse(customCatalog);
      catalog = catalog.concat(customCatalog);
    }
    return {
      denominations: [{ amt: 0, itr: 0.05 }],
      balance: 100000,
      catalog,
      initCatalogLen,
      editIndex: -1,
      SRC_LINK,
      saveFileName: "",
    };
  },
  computed: {
    isEditing() {
      return this.editIndex >= 0;
    },
    eir() {
      var a = this.totalEarned;
      var b = this.balance;
      return b > 0 ? (a / b) * 100 : 0;
    },
    dSum() {
      var ded = this.denominations;
      var i = 0;
      var sum = 0;
      var len = ded.length - 1;
      for (i = 0; i < len; i++) {
        var d = ded[i];
        sum += d.amt;
      }
      return sum;
    },
    lastAmt() {
      return this.denominations.length > 1
        ? this.dSum
        : this.denominations[0].amt;
    },
    totalEarned() {
      var ded = this.denominations;
      var i = 0;
      var len = ded.length - 1;
      var sum = 0;
      var totalBal = this.balance;
      for (i = 0; i < len; i++) {
        var d = ded[i];
        var deduct = d.amt;
        totalBal -= d.amt;
        if (totalBal < 0) deduct += totalBal;
        sum += (deduct * d.itr) / 100;
        if (totalBal <= 0) break;
      }

      var lastAmt = this.lastAmt;
      var remainingAmt = this.balance - lastAmt;
      if (remainingAmt < 0) remainingAmt = 0;
      sum += (remainingAmt * ded[ded.length - 1].itr) / 100;
      sum = parseFloat(sum.toFixed(2));
      return sum;
    },
  },
  methods: {
    onAddCatalogClick() {
      if (this.isEditing) {
        this.catalog[this.editIndex].name = this.saveFileName;
        this.catalog[this.editIndex].denom = this.deepClone(this.denominations);
      } else {
        this.catalog.push({
          name: this.saveFileName,
          editable: true,
          denom: this.deepClone(this.denominations),
        });
        this.saveFileName = "";
        this.editIndex = -1;
      }

      this.saveUserData();
    },
    saveUserData() {
      localStorage.setItem(
        "customCatalog",
        JSON.stringify(this.catalog.slice(this.initCatalogLen))
      );
    },
    activateCatalogItemAt(i) {
      this.activateCatalogItem(this.catalog[i]);
    },
    deleteItemAt(i) {
      this.catalog.splice(i, 1);
      this.editIndex = -1;
      this.saveFileName = "";
      this.saveUserData();
    },
    editItemAt(i) {
      this.editIndex = i;
      let catalogItem = this.catalog[i];
      this.saveFileName = catalogItem.name;
      this.denominations = this.deepClone(catalogItem.denom);
    },
    activateCatalogItem(c) {
      this.denominations = this.deepClone(c.denom);
      this.editIndex = -1;
      this.saveFileName = "";
    },
    deepClone(arr) {
      return JSON.parse(JSON.stringify(arr));
    },
    itrChange(i, val) {
      this.denominations[i].itr = Math.max(0, parseFloat(val));
    },
    amtChange(i, val) {
      this.denominations[i].amt = Math.max(0, parseFloat(val));
    },
    add() {
      this.denominations.push({ amt: 0, itr: 0 });
    },
    minus() {
      this.denominations.pop();
    },
  },
};
</script>

<style>
#app tbody > tr > td:first-child:before {
  content: "Next ";
}
#app tbody > tr:first-of-type > td:first-child:before {
  content: "First ";
}
#app tbody > tr:last-of-type > td:first-child:before {
  content: "Above ";
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#app h1 {
  font-size: 19px;
}
#app h2 {
  font-size: 18px;
}
#app table input {
  width: 110px;
}
#app a {
  cursor: pointer;
  text-decoration: underline;
}
#app td:last-child input {
  width: 65px;
}
#app thead td {
  font-weight: bold;
}

#app .filenamebox {
  font-size: 12px;
  padding: 5px;
}
</style>
