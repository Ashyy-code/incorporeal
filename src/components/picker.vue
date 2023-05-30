<template>
  <div class="picker-wrap">
    <div
      ref="rf_picker"
      v-if="pickerList"
      class="select-wrap"
      @click="expandPicker()"
    >
      <img :src="selPicture()" alt="" />
      <span ref="rf_sel_option">{{ pickerTitle }}</span>
      <i class="bx bxs-chevrons-down"></i>
    </div>
    <div v-show="showOptions" class="picker-options">
      <div
        v-for="option in pickerList"
        :key="option.spec_id"
        class="option"
        @click="selectOption(option)"
      >
        <span :style="'color:' + option.color" :innerText="option.name"></span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  emits: ["itemSelected"],
  methods: {
    loadPicker(data, title) {
      this.pickerList = data;
      this.pickerTitle = title;
    },

    expandPicker() {
      this.showOptions = !this.showOptions;
      this.$refs.rf_picker.toggleAttribute("expanded");
    },
    selectOption(option,filled) {
      this.selectedOption = option;
      this.showOptions = false;
      this.pickerTitle = option.name;
      if(!filled){
        this.$refs.rf_picker.toggleAttribute("expanded");
      }
      this.$refs.rf_sel_option.style = "color:" + option.color;
      this.$emit("itemSelected",option);
    },

    selPicture() {
      if (this.selectedOption != null) {
        return this.selectedOption.img;
      } else {
        return this.pickerList[0]["role_img"];
      }
    },
  },

  data() {
    return {
      pickerList: null,
      pickerTitle: null,
      selectedOption: null,
      showOptions: false,
    };
  },
};
</script>

<style lang="scss" scoped>
.picker-wrap {
  width: 300px;
  position: relative;
}
.picker-options {
  background: #1c1c1c;
  display: flex;
  flex-direction: column;
  position: fixed;
  width: 100%;
  max-width: calc(300px - 1rem);
  z-index: 9999;
  max-height: 400px;
  overflow-y: auto;

  .option {
    padding: 0.5rem;
    border-bottom: solid 1px #3b3b3b;
    cursor: pointer;
    &:hover {
      background: #3d3d3d;
    }
  }
}
.select-wrap {
  background: #1c1c1c;
  border-radius: 1rem;
  width: calc(100% - 2rem);
  padding: 0.5rem;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  margin-top: 1rem;
  cursor: pointer;

  &:hover {
    background: #3b3b3b;
  }

  &[expanded] {
    border-radius: 1rem 1rem 0 0;
  }

  img {
    height: 30px;
    width: 30px;
    border-radius: 50%;
  }
}
</style>