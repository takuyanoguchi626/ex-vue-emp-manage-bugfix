<template>
  <div class="container">
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <a class="breadcrumb">従業員リスト</a>
        </div>
      </div>
    </nav>
    <div class="row" id="searchForm">
      <div class="input-field col s12 searchForm">
        <label for="search">従業員検索</label>
        <input type="text" id="search" v-model="search" />
        <button class="searchBtn" type="button" @click="searchEmployee(search)">
          検索
        </button>
      </div>
    </div>
    <div class="searchErrorMessage">
      {{ searchErrorMessage }}
    </div>
    <div>従業員数:{{ getEmployeeCount }}人</div>
    <div class="row">
      <table class="striped">
        <thead>
          <tr>
            <th>名前</th>
            <th>入社日</th>
            <th>扶養人数</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="employee of showEmployeeList" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.formatHireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
      <table>
        <tr>
          <td v-for="pageCount of employeeListPages" :key="pageCount">
            <button type="button" @click="pageChange(pageCount)">
              {{ pageCount }}
            </button>
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Employee } from "@/types/employee";
/**
 * 従業員一覧を表示する画面.
 */
@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  private currentEmployeeList: Array<Employee> = [];
  // 従業員数
  private employeeCount = 0;
  //表示する従業員一覧
  private showEmployeeList = Array<Employee>();
  //検索内容
  private search = "";
  //検索結果が見つからなかった時のメッセージ
  private searchErrorMessage = "";

  /**
   * 従業員を検索する.
   *
   * @params - 検索内容
   */
  searchEmployee(search: string): void {
    this.searchErrorMessage = "";
    this.currentEmployeeList = this.$store.getters.getSearchEmployeeByName(
      search
    );
    if (this.currentEmployeeList.length === 0) {
      this.currentEmployeeList = this.$store.getters.getAllEmployees;
      this.searchErrorMessage = "１件もありませんでしたので全件表示します";
    }
  }

  /**
   * Vuexストアのアクション経由で非同期でWebAPIから従業員一覧を取得する.
   *
   * @remarks
   * Vueインスタンスが生成されたタイミングで
   * Vuexストア内のアクションを呼ぶ(ディスパッチする)。
   * ライフサイクルフックのcreatedイベント利用。
   *
   * 取得してからゲットするため、async awaitを利用している。
   */
  async created(): Promise<void> {
    await this.$store.dispatch("getEmployeeList");
    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.currentEmployeeList = this.$store.getters.getAllEmployees;
    this.pageChange(1);
  }
  /**
   * 現在表示されている従業員一覧の数を返す.
   *
   * @returns 現在表示されている従業員一覧の数
   */
  get getEmployeeCount(): number {
    return this.currentEmployeeList.length;
  }

  get employeeListPages(): number {
    return this.$store.getters.getEmployeeListPages;
  }

  /**
   * ページを切り替える.
   *
   * @params ページ数
   */
  pageChange(page: number): void {
    this.showEmployeeList = Array<Employee>();
    //表示するページの最初の従業員のcurrentEmployeeListの中での添え字
    let firstEmployeeOfpage = (page - 1) * 10;
    //表示するページの最後の従業員のcurrentEmployeeListの中での添え字
    let lastEmployeeOfpage = (page - 1) * 10 + 9;
    for (let i = firstEmployeeOfpage; i <= lastEmployeeOfpage; i++) {
      if (this.currentEmployeeList[i] === undefined) {
        return;
      }
      this.showEmployeeList.push(this.currentEmployeeList[i]);
    }
  }
}
</script>

<style scoped>
#searchForm {
  margin-bottom: 20px;
  width: 250px;
  margin: 0 auto;
}

.searchForm {
  display: flex;
}

.searchBtn {
  display: block;
  width: 60px;
  height: 30px;
  margin: 10px;
}

.searchErrorMessage {
  text-align: center;
}
</style>
