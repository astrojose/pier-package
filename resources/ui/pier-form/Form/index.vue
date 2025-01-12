<template>
  <form action="#" method="POST" @submit.prevent="saveRow">
    <div :key="reloadFields">
      <template v-if="model">
        <PierEditorField
          v-for="field in model.fields"
          :key="field.label"
          :field="field"
          v-model="record[field.label]"
        />
      </template>
    </div>

    <div class="mt-6 flex justify-end">
      <!-- <button
        class="p-2 text-sm mr-3"
        :class="{
          'pointer-events-none opacity-50': savingRecord || uploadingFile,
        }"
        type="reset"
        @click="$router.replace(`/${model.name}`)"
      >
        CANCEL
      </button> -->

      <button
        class="text-sm bg-blue-800 text-white py-2 px-4 rounded"
        :class="{
          'pointer-events-none opacity-50': savingRecord || uploadingFile,
        }"
        type="submit"
      >
        {{
          savingRecord
            ? `SAVING ${model.name.toUpperCase()}...`
            : "SAVE CHANGES"
        }}
      </button>
    </div>
  </form>
</template>

<script>
import { insertRecord, updateRecord } from './API';
import PierEditorField from "./PierEditorField";
import { handleNetworkError, showSuccessToast } from './Utils';

export default {
  name: "PierForm",
  mounted() {
    if (this.values) {
      this.record = this.values;
      this.reloadFields = "be best";
    }
  },
  data: function () {
    return {
      reloadFields: "be",
      record: {},
      savingRecord: false,
    };
  },
  inject: ["model", "values"],
  computed: {
    // ...mapState(["savingRecord", "model.name"]),
    // ...mapGetters(["model"]),
    modelName(){
      if(this.model) return this.model.name;

      return "";
    },
    uploadingFile: function () {
      return false;
    },
    modelFieldsTypes: function () {
      if (this.model)
        return this.model.fields.reduce((agg, { label, type }) => {
          return { ...agg, [label]: type };
        }, {});
      return null;
    },
  },
  methods: {
    async createRecord(data) {
      this.savingRecord = true;
      try {
        let record = await insertRecord(this.modelName, data);

        this.savingRecord = false;

        showSuccessToast(`${this.modelName} created`);
        if(window.pierRedirectTo && window.pierRedirectTo.length)
          window.location.href = pierRedirectTo;
      } catch (error) {
        handleNetworkError(error, `Error creating ${this.modelName}:`);
        this.savingRecord = false;
      }
    },
    async editRecord(data) {
      this.savingRecord = true;
      try {
        const record = await updateRecord(this.modelName, data);

        this.savingRecord = false;

        showSuccessToast(`${this.modelName} updated`);
        if(window.pierRedirectTo && window.pierRedirectTo.length)
          window.location.href = pierRedirectTo;
      } catch (error) {
        handleNetworkError(error, `Error creating ${this.modelName}:`);
        this.savingRecord = false;
      }
    },
    saveRow() {
      const self = this;
      const data = Object.fromEntries(
        Object.entries(this.record).map(function ([key, value]) {
          const fieldType = self.modelFieldsTypes[key];
          if (fieldType == "reference") value = value._id;
          else if (fieldType == "multi-reference")
            value = value.map(({ _id }) => _id);

          return [key, value];
        })
      );

      console.log("Data: ", data);
      if (this.values) this.editRecord(data);
      else this.createRecord(data);
    },
  },
  components: {
    PierEditorField,
  },
};
</script>