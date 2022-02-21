<template>
  <div class="container">
    <div class="row register-page">
      <form class="col s12" id="reg-form">
        <div class="row">
          <div>
            {{ errorMessage }}
          </div>
          <div class="input-field col s6">
            <input
              id="last_name"
              type="text"
              class="validate"
              v-model="lastName"
              required
            />
            <label for="last_name">姓</label>
            {{ alertLastName }}
          </div>
          <div class="input-field col s6">
            <input
              id="first_name"
              type="text"
              class="validate"
              v-model="firstName"
              required
            />
            <label for="first_name">名</label>
            {{ alertFirstName }}
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <input
              id="email"
              type="email"
              class="validate"
              v-model="mailAddress"
              required
            />
            <label for="email">メールアドレス</label>
            {{ alertEmail }}
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <input
              id="password"
              type="password"
              class="validate"
              minlength="8"
              v-model="password"
              required
            />
            <label for="password">パスワード</label>
            {{ alertPassword }}
          </div>
        </div>
        <div class="row">
          <div class="input-field col s6">
            <button
              class="btn btn-large btn-register waves-effect waves-light"
              type="button"
              v-on:click="registerAdmin"
            >
              登録
              <i class="material-icons right">done</i>
            </button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import config from "@/const/const";
import axios from "axios";

/**
 * 管理者登録をする画面.
 */
@Component
export default class RegisterAdmin extends Vue {
  // 姓
  private lastName = "";
  // 名
  private firstName = "";
  // メールアドレス
  private mailAddress = "";
  // パスワード
  private password = "";
  //入力値チェック（姓）
  private alertLastName = "";
  //入力値チェック（名）
  private alertFirstName = "";
  //入力値チェック（メールアドレス）
  private alertEmail = "";
  //入力値チェック（パスワード）
  private alertPassword = "";
  //エラーチェック
  private errorCheck = false;
  //エラーメッセージ
  private errorMessage = "";

  /**
   * 管理者情報を登録する.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出し管理者登録をするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async registerAdmin(): Promise<void> {
    this.alertLastName = "";
    this.alertFirstName = "";
    this.alertEmail = "";
    this.alertPassword = "";
    this.errorCheck = false;

    if (this.lastName === "") {
      this.alertLastName = "姓が入力されていません";
      this.errorCheck = true;
    }
    if (this.firstName === "") {
      this.alertFirstName = "名が入力されていません";
      this.errorCheck = true;
    }
    if (this.mailAddress === "") {
      this.alertEmail = "メールアドレスが入力されていません";
      this.errorCheck = true;
    }
    if (this.password === "") {
      this.alertPassword = "パスワードが入力されていません";
      this.errorCheck = true;
    }
    if (this.errorCheck === true) {
      return;
    }

    // 管理者登録処理
    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + " " + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
    });
    console.dir("response:" + JSON.stringify(response));
    const statusMessage = response.data.status;
    if (statusMessage === "error") {
      this.errorMessage = "登録できませんでした";
      return;
    }

    this.$router.push("/logoutAdmin");
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}
</style>
