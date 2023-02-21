<template>
  <div 
    class="wrapper"
  >
    <div class="top-panel">
      <div>
        <template v-if="oRepo"> 
        {{oRepo.name}}
        </template>
      </div>
      <div class="top-right-panel">
      </div>
    </div>
    <div class="app-modes">
      <button v-for="(oMenuItem, iI) in aMenu" :key="iI" class="btn" @click="fnClickLeftMenu(oMenuItem)" :title="oMenuItem.title"><i :class="'bi '+oMenuItem.icon"></i></button>
      <button class="btn btn-import" title="Импортировать"><i class="bi bi-box-arrow-in-up"></i><label><input type="file" ref="file_selector" @change="fnFileImportChange" /></label></button>
    </div>
    <div class="current-mode">
      <div class="models-tree">
        <list :form_name="'table'" :table_name="'table'" :filter_field="'name'">
          <template v-slot:default="p">
            {{ p.oItem.name }}
          </template>
        </list>
      </div>
    </div>
    <div class="page-panel" v-show="oSelectedItem">
      <iframe 
          src="./drawio.html" 
          style="width:100%;height:100%" 
          ref="drawioframe"
          name="drawioframe"
      ></iframe>
    </div>
  </div>
  <saved_toast />
  <repo_window />
  <loader/>
</template>


<script>
import repo_window from "./components/repo_window.vue"
import loader from "./components/loader.vue"
import saved_toast from "./components/saved_toast.vue"

import list from "./components/list.vue"

import { mapMutations, mapState, mapActions, mapGetters } from 'vuex'
import { a, cc } from "./lib"

export default {
  name: 'App',

  components: {
    list,
    repo_window,
    loader,
    saved_toast,
  },

  computed: {
    ...cc(`bShowEditor bShowSaveToast`),
    oDatabase: {
      get() {
          return this.$store.state.oDatabase
      },
    },
    oSelectedItem: {
      get() {
          return this.$store.state.oDatabase['table'][`selection_item`]
      },
    }
  },

  watch: {
    oSelectedItem(mN, mO) {
      this.fnIframeLoadSVG()
    }
  },

  data() {
    return {
      aMenu: [
        { id: "repo-window", title: "Выбрать репозиторий", icon: "bi-person-fill" },
        { id: "save", title: "Сохранить", icon: "bi-arrow-up-square" },
        // { id: "import", title: "Импортировать", icon: "bi-box-arrow-in-up" },
        { id: "export", title: "Экспортировать", icon: "bi-box-arrow-down" },
      ],
    }
  },

  methods: {
    ...mapMutations(a`fnLoadRepos`),
    ...mapActions(a`fnSaveDatabase fnImportDatabase fnExportDatabase`),

    fnClickLeftMenu(oItem) {
      if (oItem.id == "repo-window") {
        this.bShowRepoWindow = true
      }
      if (oItem.id == "save") {
        this.fnSaveAll()
      }
      if (oItem.id == "export") {
        this.fnExport()
      }
    },

    fnIframeLoadSVG() {
      var oData = {
        command: "load",
        data: this.oSelectedItem.svg
      }
      console.log("oData", oData)

      window.frames.drawioframe.postMessage(JSON.stringify(oData));
    },

    fnIframeSaveSVG() {
      var oData = {
        command: "save",
      }

      window.frames.drawioframe.postMessage(JSON.stringify(oData));      
    },

    fnSaveAll() {
      this.fnIframeSaveSVG()
    },

    fnExport() {
      this.fnExportDatabase()
    },
    fnImport() {
      let oFile = this.$refs.file_selector.files[0];
      let reader = new FileReader();
      var oThis = this

      reader.readAsText(oFile);

      reader.onload = function() {
        oThis.fnImportDatabase(reader.result)
      };

      reader.onerror = function() {
        console.error(reader.error);
      };
    },
    fnFileImportChange() {
      this.fnImport()
    },
  },

  // watch: {
  //   oDatabase(oN, oO) {
  //     if (this.oSelectedItem) {
  //       console.log(">>>", this.oSelectedItem)
  //       setTimeout(() => {
  //         this.fnIframeLoadSVG()
  //       }, 300)
  //     }
  //   }
  // },

  created() {
    var oThis = this

    window.addEventListener("message", (event) => {
      try {
        var oData = JSON.parse(event.data)
      
        console.log(oData)
        if (oData.command=="save") {
          oThis.oSelectedItem.svg = oData.data
          oThis.fnSaveDatabase()
          oThis.bShowSaveToast = true
        }
      } catch (_) {

      }
    }, false);

    this.fnLoadRepos()

    document.addEventListener('keydown', e => {
      if (e.ctrlKey && e.keyCode === 83) {
          e.preventDefault();
          oThis.fnSaveAll()
      }
    });

    window.addEventListener('click', (e) => {
      if (this.bShowTaskMenu) {
        if (oThis.$refs.task_menu.contains(e.target)) {

        } else {
          this.bShowTaskMenu = false
        }
      }
    })  
  }
}
</script>

<style>

</style>
