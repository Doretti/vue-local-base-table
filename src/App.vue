<template>
  <div id="app">
    <div class="table">
      <div class="table-header">
        <div class="header-info">
          <h2 class="header-title">
            Currencies info
          </h2>
          <div class="header-inputs">
            <div @click="() => edit = !edit" :class="{edit: true, input: true, activeEdit: edit}">
              Edit table
              <div v-if="edit" class="modal-edit">
                <div @click="() => showCurr[i].set = !s.set" v-for="(s, i) in showCurr" :key="s.value" class="setting">
                  <img v-if="s.set" src="./assets/open.svg" alt="">
                  <img v-else src="./assets/close.svg" alt="">
                  {{s.value}}
                </div>
              </div>
            </div>
            <div class="search input">
              <input v-model="filter" @input="setCurrFilter" placeholder="Search" type="text">
            </div>
          </div>
        </div>
      </div>
      <div class="table-content">
        <div class="table-cols">
          <div v-for="cur in showCurr.filter(cur => cur.set)" @click="() => sortCurr(cur.value)" :key="cur.value" class="table-col col-sort">
            {{cur.value}}
          </div>
        </div>
        <div v-for="cur in filtredCurr" :key="cur.index" class="table-cols">
          <div v-if="showCurr[0].set" class="table-col">
            {{cur.last_update_at}}
          </div>
          <div v-if="showCurr[1].set" class="table-col">
            {{cur.code}}
          </div>
          <div v-if="showCurr[2].set" class="table-col">
            {{cur.name}}
          </div>
          <div v-if="showCurr[3].set" class="table-col">
            {{cur.deposit_enabled}}
          </div>
          <div v-if="showCurr[4].set" class="table-col">
            {{cur.withdrawal_enabled}}
          </div>
          <div v-if="showCurr[5].set" class="table-col">
            {{cur.trading_enabled}}
          </div>
        </div>
      </div>
      <div class="table-footer">
        Total: {{filtredCurr.length}}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {},
  data() {
    return {
      curr: [],
      showCurr: [
        {
          value: 'Last update',
          set: true
        },
        {
          value: 'Code',
          set: true
        },
        {
          value: 'Fullname',
          set: true
        },
        {
          value: 'Enabled deposit',
          set: true
        },
        {
          value: 'Enabled Withdrawal',
          set: false
        },
        {
          value: 'Enabled Trading',
          set: true
        },
      ],
      filtredCurr: [],
      filter: '',
      edit: false,
      sort: false
    }
  },
  mounted() {
    const xhr = new XMLHttpRequest();

    xhr.onreadystatechange = function() {
      if (xhr.readyState == 4 && xhr.status == 200) { 
        const jsonData = JSON.parse(xhr.responseText)
        setCurr(jsonData)
      }
    };
    xhr.open("GET", 'currencies.json', true)
    xhr.send();
    const setCurr = data => {
      this.curr = data
      this.filtredCurr = data
    }
  },
  methods: {
    setCurrFilter() {
      if (this.filter && (this.showCurr[1].set || this.showCurr[2].set)) {
        this.filtredCurr = this.curr.filter(cur => {
          if (this.showCurr[1].set && this.showCurr[2].set) {
            return cur.code.search(this.filter) != -1 || cur.name.search(this.filter) != -1
          } else if (this.showCurr[1].set) {
            return cur.code.search(this.filter) != -1
          } else {
            return cur.name.search(this.filter) != -1
          }
        })
      } else {
        this.filtredCurr = this.curr
      }
    },
    
    sortCurr(set) {
      switch (set) {
        case 'Last update':
          if (this.sort != 'date_less') {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return new Date(b.last_update_at) - new Date(a.last_update_at)
            })
            this.sort = 'date_less'
          } else {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return new Date(a.last_update_at) - new Date(b.last_update_at)
            })
            this.sort = 'code_more'
          }
          break;
        case 'Code':
          if (this.sort != 'code_less') {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return a.code < b.code ? -1 : a.code > b.code ? 1 : 0
            })
            this.sort = 'code_less'
          } else {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return b.code < a.code ? -1 : b.code > a.code ? 1 : 0
            })
            this.sort = 'code_more'
          }
          break;
        case 'Fullname':
          if (this.sort != 'name_less') {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return a.name < b.name ? -1 : a.name > b.name ? 1 : 0
            })
            this.sort = 'name_less'
          } else {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return b.name < a.name ? -1 : b.name > a.name ? 1 : 0
            })
            this.sort = 'name_more'
          }
          break;

        case 'Enabled deposit':
          if (this.sort != 'deposit_less') {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return b.deposit_enabled - a.deposit_enabled
            })
            this.sort = 'deposit_less'
          } else {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return a.deposit_enabled - b.deposit_enabled
            })
            this.sort = 'deposit_more'
          }
          break;

        case 'Enabled Withdrawal':
          if (this.sort != 'withdrawal_less') {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return b.withdrawal_enabled - a.withdrawal_enabled
            })
            this.sort = 'withdrawal_less'
          } else {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return a.withdrawal_enabled - b.withdrawal_enabled
            })
            this.sort = 'withdrawal_more'
          }
          break;
        
        case 'Enabled Trading':
          if (this.sort != 'trading_less') {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return b.trading_enabled - a.trading_enabled
            })
            this.sort = 'trading_less'
          } else {
            this.filtredCurr = this.filtredCurr.slice().sort((a, b) => {
              return a.trading_enabled - b.trading_enabled
            })
            this.sort = 'trading_more'
          }
          break;
      }
    }
  }
}
</script>

<style>

  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Roboto', sans-serif;
    outline: none;
  }

  #app {
    display: flex;
    justify-content: center;
    padding: 52px 55px;
  }

  .table {
    border: 1px solid #000;
    width: 100%;
  }

  .table-header {
    padding: 18px 43px;
    border-bottom: 1px solid #000;
    display: flex;
    justify-content: center;
  }
  
  .header-info {
    display: flex;
    width: 100%;
    align-items: center;
  }

  h2.header-title {
    font-size: 24px;
    line-height: 28px;
    width: 30%;
    height: 100%;
  }

  .header-inputs {
    width: 65%;
    display: flex;
    align-items: center;
  }

  .edit {
    cursor: pointer;
    margin-right: 33px;
    justify-content: center;
    position: relative;
  }

  .input {
    width: 300px;
    border: 1px solid rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    padding: 8px 0;
  }

  .search input,
  .search input::placeholder {
    border: none;
    font-size: 18px;
    line-height: 21px;
    color: #000;
  }

  .search input::placeholder {
    text-align: center;
  }
  
  .search input {
    margin-left: 16px;
  }

  .table-cols {
    display: flex;
    border-bottom: 1px solid #000;
    height: 82px;
    align-items: center;
    justify-content: center;
    padding: 27px 0;
  }

  .table-cols:first-child .table-col:last-child {
    border-right: none;
  }

  .table-cols:first-child .table-col {
    border-right: 2px solid #000;
  }

  .table-col {
    padding: 8px 42px;
    border-right: 1px solid #000;
    width: 100%;
    display: flex;
    justify-content: center;
  }

  .table-col:last-child {
    border-right: none;
  }

  .table-footer {
    display: flex;
    justify-content: flex-end;
    padding: 10px 30px;
  }
  
  .modal-edit {
    width: 100%;
    border: 1px solid #000;
    position: absolute;
    left: 0;
    top: 36px;
    background: #fff;
  }

  .setting {
    padding: 6px 3px;
    border-bottom: 1px solid #000;
    font-size: 24px;
    line-height: 28px;
    display: flex;
    align-items: center;
  }

  .setting img {
    width: 33px;
    height: 25px;
    margin-right: 6px;
  }

  .setting:last-child {
    border-bottom: none;
  }

  .activeEdit {
    background: #D9D9D9;
  }

  .col-sort {
    cursor: pointer;
  }
</style>
