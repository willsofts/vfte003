<template>
  <DialogForm ref="dialogForm">
    <template v-slot:header>
      <h4 class="modal-title" v-if="insertMode">{{ labels.title_new }}</h4>
      <h4 class="modal-title" v-if="updateMode">{{ labels.title_edit }}</h4>
    </template>
    <template v-slot:default>
        <div class="row row-height">
          <div class="col-height col-md-5">
            <label for="product">{{ labels.product_label }}</label>
            <div class="input-group has-validation" :class="{'has-error': v$.product.$error}">
              <InputMask ref="product" v-model="localData.product" id="product" picture="(15)X" :disabled="disabledKeyField"/> 
              <label class="required">*</label>
            </div>
            <span v-if="v$.product.$error" class="has-error">{{ v$.product.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-10">
            <label for="nameen">{{ labels.nameen_label }}</label>
            <div class="input-group has-validation" :class="{'has-error': v$.nameen.$error}">
              <input type="text" ref="nameen" v-model="localData.nameen" id="nameen" class="form-control input-md" maxlength="100" /> 
              <label class="required">*</label>
            </div>
            <span v-if="v$.nameen.$error" class="has-error">{{ v$.nameen.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-10">
            <label for="nameth">{{ labels.nameth_label }}</label>
            <div class="input-group has-validation" :class="{'has-error': v$.nameth.$error}">
              <input type="text" ref="nameth" v-model="localData.nameth" id="nameth" class="form-control input-md" maxlength="100" /> 
              <label class="required">*</label>
            </div>
            <span v-if="v$.nameth.$error" class="has-error">{{ v$.nameth.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-10">
            <label for="url">{{ labels.url_label }}</label>
            <input ref="url" type="text" v-model="localData.url" id="url" class="form-control input-md" maxlength="100" />
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-3">
            <label for="seqno">{{ labels.sequence_label }}</label>
            <InputNumber ref="seqno" v-model="localData.seqno" id="seqno" /> 
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-5">
            <div class="form-check">
              <input ref="centerflag" type="checkbox" id="centerflag" :true-value="1" :false-value="0" v-model="localData.centerflag" class="form-control input-md form-check-input"/>
              <label for="centerflag" class="form-check-label">{{ labels.centerflag_label }}</label>
            </div>
          </div>
          <div class="col-height col-md-5">
            <div class="form-check">
              <input ref="verified" type="checkbox" id="verified" :true-value="1" :false-value="0" v-model="localData.verified" class="form-control input-md form-check-input"/>
              <label for="verified" class="form-check-label">{{ labels.verified_label }}</label>
            </div>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-md-5 col-height" id="iconfilelayer">
            <label class="control-label">{{ labels.iconfile_label }}</label>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-md-1"></div>
          <div class="col-md-9 col-height">
            <img id="iconfileimage" width="128px" height="128px" :src="getIconImage" alt="Product Image"/>
          </div>
        </div>
    </template>
    <template v-slot:footer>
      <button ref="savebutton" id="savebutton" class="btn btn-dark btn-sm" @click="saveClick" v-if="insertMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.save_button }}</button>
      <button ref="updatebutton" id="updatebutton" class="btn btn-dark btn-sm" @click="updateClick" v-if="updateMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.update_button }}</button>
      <button class="btn btn-dark btn-sm" data-dismiss="modal"><em class="fa fa-close fa-btn-icon"></em>{{ labels.cancel_button }}</button>
    </template>
  </DialogForm>
</template>
<script>
import { ref, computed, watch } from 'vue';
import { useVuelidate } from '@vuelidate/core';
import { required, helpers } from '@vuelidate/validators';
import $ from "jquery";
import { DEFAULT_CONTENT_TYPE, getApiUrl, getImgUrl, disableControls }  from '@willsofts/will-app';
import { startWaiting, stopWaiting, submitFailure, detectErrorResponse }  from '@willsofts/will-app';
import { confirmUpdate, confirmSave, confirmDelete, successbox, serializeParameters } from '@willsofts/will-app';
import { InputNumber, InputMask } from '@willsofts/will-control';
import DialogForm from './DialogForm.vue';

const APP_URL = "/api/sfte003";
const defaultData = {
  product: "",
  nameen: "",
  nameth: "",
  seqno: 0,
  url: "",
  centerflag: "0",
  verified: "0",
  iconfile: "",
};

export default {
  components: {
    DialogForm, InputNumber, InputMask
  },
  props: {
    modes: Object,
    labels: Object,
  },
  emits: ["data-saved","data-updated","data-deleted"],
  setup(props) {
    const mode = ref({action: "new", ...props.modes});
    const localData = ref({ ...defaultData }); 
    const disabledKeyField = ref(false);
    const reqalert = ref(props.labels.empty_alert);
    const requiredMessage = () => {
      return helpers.withMessage(reqalert, required);
    }
    const validateRules = computed(() => { 
      return {
        product: { required: requiredMessage() },
        nameen: { required: requiredMessage() },
        nameth: { required: requiredMessage() },
      } 
    });
    const v$ = useVuelidate(validateRules, localData, { $lazy: true, $autoDirty: true });
    return { mode, v$, localData, disabledKeyField, reqalert };
  },
  created() {
    watch(this.$props, (newProps) => {      
      this.reqalert = newProps.labels.empty_alert;
    });
  },
  computed: {
    insertMode() {
      return this.mode.action=="insert" || this.mode.action=="new";
    },
    updateMode() {
      return this.mode.action=="update" || this.mode.action=="edit";
    },
    getIconImage() {
      return this.localData.iconfile && this.localData.iconfile.trim().length > 0 ? getImgUrl()+"/img/module/"+this.localData.iconfile : getImgUrl()+"/img/module/module.png";
    },
  },
  mounted() {
    this.$nextTick(function () {
      $("#modaldialog_layer").find(".modal-dialog").draggable();
    });
  },
  methods: {
    reset(newData,newMode) {
      if(newData) this.localData = {...newData};
      if(newMode) this.mode = {...newMode};
    },
    submit() {      
      this.$emit('update:formData', this.localData);
    },
    async saveClick() {
      console.log("click: save");
      disableControls($("#savebutton"));
      let valid = await this.validateForm();
      if(!valid) return;
      this.startSaveRecord();
    },
    async updateClick() {
      console.log("click: update");
      disableControls($("#updatebutton"));
      let valid = await this.validateForm();
      if(!valid) return;
      this.startUpdateRecord();
    },
    async validateForm(focusing=true) {
      const valid = await this.v$.$validate();
      console.log("validate form: valid",valid);
      console.log("error:",this.v$.$errors);
      if(!valid) {
        if(focusing) {
          this.focusFirstError();
        }
        return false;
      }
      return true;
    },
    focusFirstError() {
      if(this.v$.$errors && this.v$.$errors.length > 0) {
        let input = this.v$.$errors[0].$property;
        let el = this.$refs[input];
        if(el) el.focus(); //if using ref
        else $("#"+input).trigger("focus"); //if using id
      }
    },
    showDialog() {
      //$("#modaldialog_layer").modal("show");
      $(this.$refs.dialogForm.$el).modal("show");
    },  
    hideDialog() {
      //$("#modaldialog_layer").modal("hide");
      $(this.$refs.dialogForm.$el).modal("hide");
    },
    resetRecord() {
      //reset to default data 
      this.reset(defaultData,{action:"insert"});
      //reset validator
      this.v$.$reset();
      //enable key field
      this.disabledKeyField = false;
    },
    startInsertRecord() {
      this.resetRecord();
      this.showDialog();
    },
    startSaveRecord() {
      confirmSave(() => {
        this.saveRecord(this.localData);
      });
    },
    startUpdateRecord() {
      confirmUpdate(() => { 
        this.updateRecord(this.localData);
      });
    },
    startDeleteRecord(dataKeys) {
      confirmDelete(Object.values(dataKeys),() => {
        this.deleteRecord(dataKeys);
      });
    },
    saveRecord(dataRecord) {
        let jsondata = {ajax: true};
        let formdata = serializeParameters(jsondata,dataRecord);
        startWaiting();
        $.ajax({
          url: getApiUrl()+APP_URL+"/insert",
          data: formdata.jsondata,
          headers : formdata.headers,
          type: "POST",
          dataType: "json",
          contentType: DEFAULT_CONTENT_TYPE,
          error : function(transport,status,errorThrown) {
            console.error("error: status",status,"errorThrown",errorThrown);
            submitFailure(transport,status,errorThrown);
          },
          success: (data) => {
            stopWaiting();
            console.log("success",data);
            if(detectErrorResponse(data)) return;
            successbox(() => {
              //reset data for new record insert
              this.resetRecord();
              setTimeout(() => { this.$refs.product.focus(); },100);
            });
            this.$emit('data-saved',dataRecord,data);
          }
      });
    },
    updateRecord(dataRecord) {
        let jsondata = {ajax: true};
        let formdata = serializeParameters(jsondata,dataRecord);
        startWaiting();
        $.ajax({
          url: getApiUrl()+APP_URL+"/update",
          data: formdata.jsondata,
          headers : formdata.headers,
          type: "POST",
          dataType: "json",
          contentType: DEFAULT_CONTENT_TYPE,
          error : function(transport,status,errorThrown) {
            console.error("error: status",status,"errorThrown",errorThrown);
            submitFailure(transport,status,errorThrown);
          },
          success: (data) => {
            stopWaiting();
            console.log("success",data);
            if(detectErrorResponse(data)) return;
            successbox(() => {
              this.hideDialog();
            });
            this.$emit('data-updated',dataRecord,data);
          }
      });
    },
    retrieveRecord(dataKeys) {
      let jsondata = {ajax: true};
      let formdata = serializeParameters(jsondata,dataKeys);
      startWaiting();
      $.ajax({
        url: getApiUrl()+APP_URL+"/retrieve",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("retrieveRecord: error: status",status,"errorThrown",errorThrown);
          submitFailure(transport,status,errorThrown);
        },
        success: (data) => {
          stopWaiting();
          console.log("retrieveRecord: success",data);
          if(data.body.dataset) {
            this.reset(data.body.dataset,{action:"edit"});
            this.v$.$reset();
            this.disabledKeyField = true;
            this.showDialog();
          }
        }
      });	
    },
    deleteRecord(dataKeys) {
      let jsondata = {ajax: true};
      let formdata = serializeParameters(jsondata,dataKeys);
      startWaiting();
      $.ajax({
        url: getApiUrl()+APP_URL+"/remove",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("deleteRecord: error: status",status,"errorThrown",errorThrown);
          submitFailure(transport,status,errorThrown);
        },
        success: (data) => {
          stopWaiting();
          console.log("deleteRecord: success",data);
          if(detectErrorResponse(data)) return;
          successbox();
          this.$emit('data-deleted',dataKeys,data);
        }
      });	
    },
  }
};
</script>
