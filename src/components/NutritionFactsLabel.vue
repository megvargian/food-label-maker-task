<template>
  <div v-if="!loading">
    <div class="border border-black p-2 max-w-[17rem] mx-auto text-balck" :dir="isRTL" :style="`font-family: ${isRTL === 'rtl' ? 'arabic' : 'roboto'}`">
      <h1 class="font-black text-3xl"> {{ isRTL === 'rtl' ? 'حقائق التغذية': 'Nutrition Facts'}}</h1>
      <div class="border-t border-black"></div>
      <p class="text-sm"
        :class="isRTL === 'rtl' ? 'text-right' : 'text-left'">
        <strong>{{ data.label.amounts.number_of_servings }}</strong> {{ isRTL === 'rtl' ? 'الحصص لكل عبوة': 'Serving Per Container'}}
      </p>
      <div class="flex justify-between items-center font-black text-base">
        <h5> {{ isRTL === 'rtl' ? 'حجم الحصة': 'Serving Size'}}</h5>
        <p>{{ data.label.amounts.serving }}g</p>
      </div>
      <div class="border-t-[1rem] border-black"></div>
      <div class="flex justify-between items-center font-black">
        <div :class="isRTL === 'rtl' ? 'text-right' : 'text-left'">
          <h6 class="text-sm block">{{ isRTL === 'rtl' ? 'الكمية لكل حصة': 'Amount Per Serving'}}</h6>
          <h3 class="text-2xl">{{ isRTL === 'rtl' ? "السعرات الحرارية": 'Calories'}}</h3>
        </div>
        <p class="text-4xl">
          {{ Math.round(data.label.serving.Calories.value) }}
        </p>
      </div>
      <div class="border-t-[.5rem] mt-2 border-black"></div>
      <div class="text-xs">
        <div class="justify-end flex items-center font-black py-1">
          <p class="text-xs">%  {{ isRTL === 'rtl' ? 'القيمة اليومية': 'Daily Value'}} * </p>
        </div>
        <div class="border-t border-black"></div>
        <div
          v-for="(serving, index) in sortSectionOrderLabelNutrientsArray"
          :key="serving.id"
        >
          <div v-if="serving.enabled === 1">
            <div class="justify-between flex items-center py-1">
              <p
                :class="
                  serving.indentations === 1
                    ? `p${isRTL === 'rtl' ? 'r' : 'l'}-${serving.indentations + 2}`
                    : serving.indentations > 1
                    ? `p${isRTL === 'rtl' ? 'r' : 'l'}-${serving.indentations + 3}`
                    : ''
                "
              >
                <strong
                  :class="serving.indentations === 0 ? 'font-black' : ''"
                  >{{ isRTL === 'rtl' ? serving.name_ar : serving.name }}</strong
                >
                {{ Math.round(serving.value) }}
                {{
                  typeof serving.unit === "object" && serving.unit !== null
                    ? serving.unit.name
                    : serving.unit === null
                    ? "g"
                    : ""
                }}
              </p>
              <p v-if="data.label.daily_value[serving.name]" class="font-black">
                {{ Math.round(data.label.daily_value[serving.name]) }}%
              </p>
            </div>
            <div
              class="border-black"
              :class="
                index > 0 &&
                serving.section >
                  sortSectionOrderLabelNutrientsArray[index - 1].section
                  ? 'border-t-[.6rem]'
                  : 'border-t'
              "
            ></div>
          </div>
        </div>
        <div class="border-t-[.5rem] border-black"></div>
        <div
        class="text-[.7rem]"
        :class="isRTL === 'rtl' ? 'text-right' : 'text-left'">
          {{  isRTL === 'rtl' ? 'النسبة المئوية للقيمة اليومية (%DV) تخبرك بكمية المغذيات في حصة الطعام التي تساهم في النظام الغذائي اليومي. يُستخدم 2000 سعر حراري يوميًا كمرجع عام للتغذية.' : '* the % Dailly Value (DV) tells you how much a nutrient in a serving of foof cotributes to a daily diet 2,000 calories a day is used for general nutrition advice.'}}
        </div>
      </div>
    </div>
    <div class="checkboxes">
      <div class="toggle-container flex justify-between items-center">
        <p>Toggle LTR/RTL</p>
        <label class="switch">
          <input type="checkbox" @change="changeDirections()"/>
          <span class="slider"></span>
        </label>
      </div>
      <div v-for="(serving) in sortSectionOrderLabelNutrientsArray" :key="serving.id">
          <div class="toggle-container flex justify-between items-center">
              <p>{{ serving.name }}</p>
              <label class="switch">
                <input type="checkbox" v-model="serving.enabled" @change="showNutrient(serving.name)"/>
                <span class="slider"></span>
              </label>
            </div>
          </div>
      </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isRTL: 'ltr',
      data: {},
      loading: false,
    };
  },
  async created() {
    await this.fetchAllData();
  },
  computed: {
    sortSectionOrderLabelNutrientsArray() {
      return Object.entries(this.data.label.serving)
        .map(([key, value]) => ({ key, ...value })) // added a key for each value
        .sort((serving1, serving2) => {
          if (serving1.section !== serving2.section) {
            return serving1.section - serving2.section;
          }
          return serving1.order - serving2.order;
        });
    },
  },
  methods: {
    showNutrient(Name){
      const originalServing = Object.values(this.data.label.serving).find(serving => serving.name === Name);
      if (originalServing) {
        originalServing.enabled = originalServing.enabled === 1 ? 0 : 1;
      }
    },
    changeDirections(){
      this.isRTL === 'ltr' ? this.isRTL = 'rtl' : this.isRTL = 'ltr';
    },
    async fetchAllData() {
      try {
        // fetch the mock data response.json form public folder
        this.loading = true;
        const response = await fetch('/response.json');
        if (!response.ok) {
          throw new Error('Failed to fetch data');
        }
        this.data = await response.json();
        this.loading = false;
      } catch (err) {
        console.error(err);
      }
    }
  },
};
</script>
<style scoped>
.toggle-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: 0.4s;
  border-radius: 34px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 26px;
  width: 26px;
  border-radius: 50%;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: 0.4s;
}

input:checked + .slider {
  background-color: #4CAF50;
}

input:checked + .slider:before {
  transform: translateX(26px);
}
</style>