<template>
  <div class="input">
    <div class="input_header">
      <router-link class="arrow" to="/">
        <img src="../assets/img/arrow.png" />
      </router-link>

      <h2>手動輸入發票</h2>
    </div>
    <div class="input_contain">
      <div class="input_contain_items">
        <div class="input_items">
          <span class="input_box input_text">發票號碼:</span>
          <div class="input_box">
            <div class="input_box_input">
              <input
                placeholder="大寫英文"
                maxlength="2"
                v-model="invoiceCode"
                @input="validateInvoiceCode"
                :class="invCodeIsValid ? '' : 'input_warning'"
              />
              <div class="input_tip" v-if="!invCodeIsValid">
                請輸入正確的發票代碼
              </div>
            </div>
            <div class="input_box_input">
              <input
                maxlength="8"
                placeholder="8碼發票號碼"
                v-model="invoiceNum"
                @input="validatInvoiceNum"
                :class="invNumIsValid ? '' : 'input_warning'"
              />
              <div class="input_tip" v-if="!invNumIsValid">
                請輸入正確的發票號碼
              </div>
            </div>
          </div>
        </div>
        <div class="input_items">
          <span class="input_box input_text">開立時間:</span>
          <div class="input_box">
            <div class="input_box_input">
              <input
                maxlength="4"
                placeholder="西元年"
                v-model="year"
                @input="validatYear"
                :class="validatMoment() || year === '' ? '' : 'input_warning'"
              />
            </div>
            <div class="input_box_input">
              <input
                maxlength="2"
                placeholder="月份"
                v-model="month"
                @input="validatMonth"
                @blur="padStart($event, 'month')"
                :class="validatMoment() || month === '' ? '' : 'input_warning'"
              />

              <input
                maxlength="2"
                placeholder="日期"
                v-model="date"
                @input="validatDate"
                @blur="padStart($event, 'date')"
                :class="validatMoment() || date === '' ? '' : 'input_warning'"
              />
            </div>
          </div>
          <div
            class="input_tip_date"
            v-if="
              !validatMoment() && !(year === '' && month === '' && date === '')
            "
          >
            請輸入正確的日期
          </div>
        </div>
      </div>
      <button class="cursor-pointer" type="submit" @click="submit">提交</button>
    </div>
  </div>
</template>
<script>
import * as moment from "moment";
import axios from "axios";
export default {
  name: "Input",
  components: {},
  data() {
    return {
      invoiceCode: "",
      invoiceNum: "",
      year: "",
      month: "",
      date: "",
    };
  },
  methods: {
    //英文字母兩碼
    //輸入小寫字母可轉成大寫字母
    //輸入內容不符合時，無法輸入
    validateInvoiceCode() {
      this.invoiceCode = this.invoiceCode.toUpperCase();
      const regex = /^[A-Z]{1,2}$/g;
      const pass = regex.test(this.invoiceCode);
      if (!pass) {
        this.invoiceCode = "";
      }
    },
    //驗證 發票號碼八碼數字
    validatInvoiceNum() {
      const regex = /^[0-9]{1,8}$/g;
      const pass = regex.test(this.invoiceNum);
      if (!pass) {
        this.invoiceNum = "";
      }
    },
    //驗證 西元年
    validatYear() {
      const regex = /^[0-9]{1,4}$/g;
      const pass = regex.test(this.year);
      if (!pass) {
        this.year = "";
      }
      this.validatMoment();
    },
    //驗證 月份
    validatMonth() {
      const regex = /^[0-9]{1,2}$/g;
      const pass = regex.test(this.month);
      if (!pass) {
        this.month = "";
      }
      this.validatMoment();
    },
    //驗證 日期
    validatDate() {
      const regex = /^[0-9]{1,2}$/g;
      const pass = regex.test(this.date);
      if (!pass) {
        this.date = "";
      }
      this.validatMoment();
    },

    //驗證 輸入的 西元年/月/日 是否為有效
    validatMoment() {
      let result = moment(
        `${this.month}/${this.date}/${this.year}`,
        "MM/DD/YYYY",
        true
      ).isValid();
      return result;
    },
    //驗證完整發票號碼
    validatInvnum() {
      const regex = /^[A-Z]{2}?[0-9]{8}$/g;
      const result = regex.test(this.invoiceData.invNum);
      return result;
    },
    padStart(e, key) {
      const oldV = e.target.value;
      const newV = oldV.padStart(2, "0");
      this[key] = newV;
    },
    async getinvoicesList() {
      await axios
        .post(`http://localhost:3000/invoices`, this.invoiceData)
        .then((res) => {
          console.log(res);
        })
        .catch((err) => {
          console.log(err);
        });
    },

    submit() {
      console.log("validatMoment:" + this.validatMoment());
      console.log("validatInvnum:" + this.validatInvnum());
      if (this.validatMoment() && this.validatInvnum()) {
        this.getinvoicesList();
        sessionStorage.setItem("maxID", this.invoiceData.id);
        this.resetInput();
      }
    },
    resetInput() {
      this.invoiceCode = "";
      this.invoiceNum = "";
      this.year = "";
      this.month = "";
      this.date = "";
    },
  },

  computed: {
    invoiceData() {
      let maxID = JSON.parse(sessionStorage.getItem("maxID"));
      const newMaxID = (maxID += 1);
      return {
        id: newMaxID,
        invNum: `${this.invoiceCode}${this.invoiceNum}`,
        status: "驗證中",
        time: `${this.year}-${this.month}-${this.date} 00:00:00`,
        type: 1,
      };
    },
    invCodeIsValid() {
      const regex = /^[A-Z]{2}$/g;
      const result = regex.test(this.invoiceCode);
      return result || this.invoiceCode === "";
    },
    invNumIsValid() {
      const regex = /^[0-9]{8}$/g;
      const result = regex.test(this.invoiceNum);
      return result || this.invoiceNum === "";
    },
  },
};
</script>

<style lang="scss" scoped>
@import "../assets/scss/input.scss";
</style>
