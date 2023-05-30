<template>
  
  <img class="logo" src="./assets/incorp.png" alt="" />
  <h1 ttl>Is it my job to worry about Incorporeal Mobs?</h1>

  <div class="picker-wrapper">
    <picker ref="rf_tank" @item-selected="selectItem('tank')" />
    <picker ref="rf_damage1" @item-selected="selectItem('dmg1')" />
    <picker ref="rf_damage2" @item-selected="selectItem('dmg2')" />
    <picker ref="rf_damage3" @item-selected="selectItem('dmg3')" />
    <picker ref="rf_healer" @item-selected="selectItem('healer')" />
    <button @click="suggestFriends()"><img src="./assets/thonk.png" /></button>
    <button cl @click="clearAll()"><i class='bx bxs-trash' ></i></button>
  </div>

  <div v-if="sortedOptions.length > 0" class="output" thn>
    <div class="rating">
      <h2>Your Group Rating for Incorporeal:</h2>
      <h2 :style="getRatingStyle()">{{ groupRating }}</h2>
    </div>
    <h1 :style="getRatingStyle()">{{ groupRatingDesc }}</h1>
  </div>

  <div v-if="sortedOptions.length > 0" class="output">
    <div class="top-prio">
      <h2>Priority 1</h2>
      <img :src="sortedOptions[0]['img']" alt="" />
      <h2
        :style="'color:' + sortedOptions[0]['color']"
        :innerText="sortedOptions[0]['name']"
      ></h2>
      <h3>
        Gets first Incorporeal with:<br />
        <b :style="'color:' + sortedOptions[0]['color']">{{
          sortedOptions[0]["spell_1"]
        }}</b>
      </h3>
    </div>
    <div class="top-prio">
      <h2>Priority 2</h2>
      <img :src="sortedOptions[1]['img']" alt="" />
      <h2
        :style="'color:' + sortedOptions[1]['color']"
        :innerText="sortedOptions[1]['name']"
      ></h2>
      <h3>
        Gets second Incorporeal with:<br />
        <b :style="'color:' + sortedOptions[1]['color']">{{
          sortedOptions[1]["spell_1"]
        }}</b>
      </h3>
    </div>
    <div class="top-prio" bk>
      <h2>Backup</h2>
      <img :src="sortedOptions[2]['img']" alt="" />
      <h2
        :style="'color:' + sortedOptions[2]['color']"
        :innerText="sortedOptions[2]['name']"
      ></h2>
      <h3>
        Is getting any missed with:<br />
        <b :style="'color:' + sortedOptions[2]['color']">{{
          sortedOptions[2]["spell_1"]
        }}</b>
      </h3>
    </div>
  </div>

  <div v-if="sortedOptions.length > 0" class="output">
    <table>
      <thead>
        <tr>
          <td>Priority</td>
          <td>Player</td>
          <td>Are Incorporeals your job?</td>
          <td>Main spell</td>
          <td>In a pinch?</td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(option, index) in sortedOptions" :key="option.spec_id">
          <td>{{ index + 1 }}</td>
          <td>{{ option.name }}</td>
          <td>{{ option.is_job }}</td>
          <td>{{ option.spell_1 }}</td>
          <td>{{ option.spell_2 }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import picker from "./components/picker.vue";
export default {
  components: { picker },
  mounted() {
    this.loadSpecList().then(() => {
      this.$refs.rf_tank.loadPicker(this.tankList, "Tank");
      this.$refs.rf_damage1.loadPicker(this.damageList, "Damage");
      this.$refs.rf_damage2.loadPicker(this.damageList, "Damage");
      this.$refs.rf_damage3.loadPicker(this.damageList, "Damage");
      this.$refs.rf_healer.loadPicker(this.healerList, "Healer");
    });
  },

  methods: {
    async loadSpecList() {
      await fetch("/specs_tm.json").then((res) =>
        res.json().then((data) => {
          this.specList = data;
          //populate the tankList
          this.specList.forEach((item) => {
            if (item.role == "Tank") {
              this.tankList.push(item);
            }
            if (item.role == "Damage") {
              this.damageList.push(item);
            }
            if (item.role == "Healer") {
              this.healerList.push(item);
            }
          });
        })
      );
    },

    getRatingStyle() {
      if (this.groupRating < 130) {
        this.groupRatingDesc = "Uh oh.. Remeber you can cycle kicks";
        return "color:#c41e3a";
      }
      if (this.groupRating >= 130 && this.groupRating < 250) {
        this.groupRatingDesc = "Everyone.. eyes open";
        return "color:#0965aa";
      }
      if (this.groupRating >= 250 && this.groupRating < 450) {
        this.groupRatingDesc = "A Total breeze";
        return "color:#bc0fb3";
      }
      if (this.groupRating >= 450) {
        this.groupRatingDesc = "A non-affix.. enjoy";
        return "color:#efb310";
      }
    },

    selectItem(name) {
      switch (name) {
        case "tank":
          this.selectedTank = this.$refs.rf_tank.selectedOption;
        case "dmg1":
          this.selectedDamage1 = this.$refs.rf_damage1.selectedOption;
        case "dmg2":
          this.selectedDamage2 = this.$refs.rf_damage2.selectedOption;
        case "dmg3":
          this.selectedDamage3 = this.$refs.rf_damage3.selectedOption;
        case "healer":
          this.selectedHealer = this.$refs.rf_healer.selectedOption;
      }

      this.selectedOptionsAll = [];
      this.groupRating = 0;

      if (
        this.selectedTank &&
        this.selectedDamage1 &&
        this.selectedDamage2 &&
        this.selectedDamage3 &&
        this.selectedHealer
      ) {
        this.selectedOptionsAll.push(this.selectedTank);
        this.selectedOptionsAll.push(this.selectedDamage1);
        this.selectedOptionsAll.push(this.selectedDamage2);
        this.selectedOptionsAll.push(this.selectedDamage3);
        this.selectedOptionsAll.push(this.selectedHealer);

        this.showOutput();
      }
    },

    showOutput() {
      //clear initial
      this.groupRating = 0;
      this.sortedOptions = this.selectedOptionsAll.sort((p1, p2) =>
        p1.spell_1_rating < p2.spell_1_rating
          ? 1
          : p1.spell_1_rating > p2.spell_1_rating
          ? -1
          : 0
      );
      console.log(this.sortedOptions);
      this.sortedOptions.forEach((option) => {
        this.groupRating += option.spell_1_rating + option.spell_2_rating;
      });
    },

    clearAll(){
      //reset all
      window.location = "index.html";
    },

    suggestFriends() {
      //sort the entire spec list by power rating for this affix
      this.sortedOptionsAll = this.specList.sort((p1, p2) =>
        p1.spell_1_rating < p2.spell_1_rating
          ? 1
          : p1.spell_1_rating > p2.spell_1_rating
          ? -1
          : 0
      );

      //blank out the current
      this.builderOptions = [];
      this.groupRating = 0;

      //check the needed classes
      let needTank = false;
      let needDPS1 = false;
      let needDPS2 = false;
      let needDPS3 = false;
      let needHealer = false;

      //builder
      let bestTank;
      let bestHealer;
      let bestDPS1;
      let bestDPS2;
      let bestDPS3;

      //add the current selected to the list
      if (this.selectedTank) {
        this.builderOptions.push(this.selectedTank);
        bestTank = this.selectedTank;
      } else {
        needTank = true;
      }
      if (this.selectedDamage1) {
        this.builderOptions.push(this.selectedDamage1);
        bestDPS1 = this.selectedDamage1;
      } else {
        needDPS1 = true;
      }
      if (this.selectedDamage2) {
        this.builderOptions.push(this.selectedDamage2);
        bestDPS2 = this.selectedDamage2;
      } else {
        needDPS2 = true;
      }
      if (this.selectedDamage3) {
        this.builderOptions.push(this.selectedDamage3);
        bestDPS3 = this.selectedDamage3;
      } else {
        needDPS3 = true;
      }
      if (this.selectedHealer) {
        this.builderOptions.push(this.selectedHealer);
        bestHealer = this.selectedHealer;
      } else {
        needHealer = true;
      }

      //check whats missing
      //console.log(this.builderOptions);

      this.buildGroupHasLust = 0;
      this.buildGroupHasPi = 0;
      this.buildGroupHasCR = 0;

      this.builderOptions.forEach((spec) => {
        this.buildGroupHasLust += parseInt(spec.has_lust);
        this.buildGroupHasPi += parseInt(spec.has_pi);
        this.buildGroupHasCR += parseInt(spec.has_cr);
      });

      //see what specs are missing

      //check tank
      if (needTank) {
        //check if CR is needed
        if (!this.buildGroupHasCR) {
          bestTank = this.sortedOptionsAll.find(
            (spec) => spec.role == "Tank" && spec.has_cr == 1
          );
          //we got a CR now
          this.buildGroupHasCR = true;
        } else {
          bestTank = this.sortedOptionsAll.find((spec) => spec.role == "Tank");
        }
        //console.log(bestTank);
      }

      //check healer
      if (needHealer) {
        //check if CR is needed
        if (!this.buildGroupHasCR) {
          bestHealer = this.sortedOptionsAll.find(
            (spec) => spec.role == "Healer" && spec.has_cr == 1
          );
          //we got CR now
          this.buildGroupHasCR = true;
        } else {
          //check if Lust is needed
          if (!this.buildGroupHasLust) {
            bestHealer = this.sortedOptionsAll.find(
              (spec) => spec.role == "Healer" && spec.has_lust == 1
            );
            //we got lust now
            this.buildGroupHasLust = true;
          } else {
            //check if PI is needed
            if (!this.buildGroupHasPi) {
              bestHealer = this.sortedOptionsAll.find(
                (spec) => spec.role == "Healer" && spec.has_pi == 1
              );
              //we got PI now
              this.buildGroupHasPi = true;
            } else {
              //take the top healer
              bestHealer = this.sortedOptionsAll.find(
                (spec) => spec.role == "Healer"
              );
            }
          }
        }
        //console.log(bestHealer);
      }

      //check DPS
      if (needDPS1) {
        if (!this.buildGroupHasCR) {
          bestDPS1 = this.sortedOptionsAll.find(
            (spec) => spec.role == "Damage" && spec.has_cr == 1
          );
          //we got CR now
          this.buildGroupHasCR = true;
        } else {
          if (!this.buildGroupHasLust) {
            bestDPS1 = this.sortedOptionsAll.find(
              (spec) => spec.role == "Damage" && spec.has_lust == 1
            );
            //we got lust now
            this.buildGroupHasLust = true;
          } else {
            if (!this.buildGroupHasPi) {
              bestDPS1 = this.sortedOptionsAll.find(
                (spec) => spec.role == "Damage" && spec.has_pi == 1
              );
              //we got PI now
              this.buildGroupHasPi = true;
            } else {
              bestDPS1 = this.sortedOptionsAll.find(
                (spec) =>
                  spec.role == "Damage" &&
                  spec.class_id != bestDPS2?.class_id &&
                  spec.class_id != bestDPS3?.class_id
              );
            }
          }
        }

        //console.log(bestDPS1);
      }

      //check DPS2
      if (needDPS2) {
        if (!this.buildGroupHasCR) {
          bestDPS2 = this.sortedOptionsAll.find(
            (spec) => spec.role == "Damage" && spec.has_cr == 1
          );
          //we got CR now
          this.buildGroupHasCR = true;
        } else {
          if (!this.buildGroupHasLust) {
            bestDPS2 = this.sortedOptionsAll.find(
              (spec) => spec.role == "Damage" && spec.has_lust == 1
            );
            //we got lust now
            this.buildGroupHasLust = true;
          } else {
            if (!this.buildGroupHasPi) {
              bestDPS2 = this.sortedOptionsAll.find(
                (spec) => spec.role == "Damage" && spec.has_pi == 1
              );
              //we got PI now
              this.buildGroupHasPi = true;
            } else {
              bestDPS2 = this.sortedOptionsAll.find(
                (spec) =>
                  spec.role == "Damage" &&
                  spec.class_id != bestDPS1?.class_id &&
                  spec.class_id != bestDPS3?.class_id
              );
            }
          }
        }

        //console.log(bestDPS2);
      }

      //check DPS3
      if (needDPS3) {
        if (!this.buildGroupHasCR) {
          bestDPS3 = this.sortedOptionsAll.find(
            (spec) => spec.role == "Damage" && spec.has_cr == 1
          );
          //we got CR now
          this.buildGroupHasCR = true;
        } else {
          if (!this.buildGroupHasLust) {
            bestDPS3 = this.sortedOptionsAll.find(
              (spec) => spec.role == "Damage" && spec.has_lust == 1
            );
            //we got lust now
            this.buildGroupHasLust = true;
          } else {
            if (!this.buildGroupHasPi) {
              bestDPS3 = this.sortedOptionsAll.find(
                (spec) => spec.role == "Damage" && spec.has_pi == 1
              );
              //we got PI now
              this.buildGroupHasPi = true;
            } else {
              bestDPS3 = this.sortedOptionsAll.find(
                (spec) =>
                  spec.role == "Damage" &&
                  spec.class_id != bestDPS2?.class_id &&
                  spec.class_id != bestDPS1?.class_id
              );
            }
          }
        }

        //console.log(bestDPS3);
      }
      //set em
      this.selectedDamage1 = bestDPS1;
      this.selectedDamage2 = bestDPS2;
      this.selectedDamage3 = bestDPS3;
      this.selectedHealer = bestHealer;
      this.selectedTank = bestTank;
      this.selectedOptionsAll.push(this.selectedTank);
      this.selectedOptionsAll.push(this.selectedDamage1);
      this.selectedOptionsAll.push(this.selectedDamage2);
      this.selectedOptionsAll.push(this.selectedDamage3);
      this.selectedOptionsAll.push(this.selectedHealer);

      this.$refs.rf_tank.selectOption(bestTank,true);
      this.$refs.rf_damage1.selectOption(bestDPS1,true);
      this.$refs.rf_damage2.selectOption(bestDPS2,true);
      this.$refs.rf_damage3.selectOption(bestDPS3,true);
      this.$refs.rf_healer.selectOption(bestHealer,true);

      this.showOutput();
    },
  },

  data() {
    return {
      specList: [],
      sortedOptionsAll: [],
      tankList: [],
      damageList: [],
      healerList: [],

      selectedTank: null,
      selectedDamage1: null,
      selectedDamage2: null,
      selectedDamage3: null,
      selectedHealer: null,

      selectedOptionsAll: [],
      sortedOptions: [],
      groupRating: 0,
      groupRatingDesc: null,

      builderOptions: [],
      buildGroupHasLust: 0,
      buildGroupHasPi: 0,
      buildGroupHasCR: 0,
    };
  },
};
</script>

<style lang="scss">
/* ===== Scrollbar CSS ===== */
/* Firefox */
* {
  scrollbar-width: auto;
  scrollbar-color: #2e2e2e rgb(107, 107, 107);
}

/* Chrome, Edge, and Safari */
*::-webkit-scrollbar {
  width: 16px;
}

*::-webkit-scrollbar-track {
  background: rgb(107, 107, 107);
}

*::-webkit-scrollbar-thumb {
  background-color: #2e2e2e;
  border-radius: 10px;
  border: 3px solid rgb(107, 107, 107);
}

body {
  background: #2e2e2e;
}

#app {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  font-size: 1rem;
  background: #2e2e2e;
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  h1[ttl],
  h2[ttl] {
    padding: 0;
    margin: 0.5rem;
    margin-bottom: 1rem;
  }

  .picker-wrapper {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    width: 100%;
    max-width: 900px;
    justify-content: center;
    align-items: center;

    button {
      background: #a5792a;
      border: 0;
      border-radius: 50%;
      padding: 0.5rem;
      color: white;
      margin-top: 1rem;
      cursor: pointer;
      text-transform: uppercase;
      font-size: 70%;
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 0.5rem;
      position: relative;

      &[cl]{
        background:#7b1515;
        padding:.75rem;
        margin-left:1rem;

        &:hover{
          background:#9c2020;
        }
        i{
          font-size:200%;

        }
      }

      &:hover {
        background: #c2933b;
      }

      img {
        height: 30px;
        width:30px;
      }
    }
  }
}
.output {
  display: flex;
  flex-direction: row;
  gap: 6rem;
  color: #9b9b9b;
  margin-top: 2rem;
  background: #0b0b0b;
  padding: 1.25rem;
  border-radius: 1rem;
  width: 900px;
  justify-content: center;

  &[thn] {
    padding-top: 0;
    flex-direction: column;
    gap: 0;

    h1 {
      padding: 0;
      margin: 0;
      text-align: center;
    }
  }

  .top-prio {
    font-size: 80%;
    width: 33.3%;
    padding: 0.25rem;

    &[bk] {
      background: #141414;
      border-radius: 0.5rem;
    }

    h2,
    h3 {
      padding: 0;
      margin: 0;
      text-align: center;
    }

    img {
      border-radius: 50%;
    }

    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    justify-content: center;
    align-items: center;
  }
}
.rating {
  display: flex;
  flex-direction: row;
  text-align: center;
  justify-content: center;
  width: 100%;

  h2 {
    padding: 0;
    margin: 1rem;
  }
}

table {
  width: 100%;
  font-size: 90%;
  color: rgb(225, 225, 225);
  border-collapse: collapse;
  td {
    border: solid 1px rgb(54, 54, 54);
    padding: 0.25rem;
  }
  thead {
    td {
      border: none;
      color: orange;
      font-size: 90%;
    }
  }
}
.logo {
  border-radius: 50%;
  margin-top: 1rem;
  height:40px;
  width:40px;


}
</style>