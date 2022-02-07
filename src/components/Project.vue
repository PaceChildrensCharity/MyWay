<template>
  <b-container fluid>
    <LongPressCollapse
      ref="fileManagerToolbarCollapse"
      :enabled="project.userMode"
    >
      <!-- Top toolbar -->
      <b-img
        src="logo-small.png"
        alt="My way: communicate, create, explore"
        left
      ></b-img>
      <b-navbar class="mt-2">
        <b-navbar-nav class="ml-auto">
          <FileManagerToolbar
            class="mr-3"
            v-bind:project="project"
            ref="fileManagerToolbar"
          ></FileManagerToolbar>
          <div class="m-1">
            <b-form-checkbox
              @change="toggleUserMode"
              v-model="project.userMode"
              switch
            >
              User mode
            </b-form-checkbox>
          </div>
        </b-navbar-nav>
      </b-navbar>
    </LongPressCollapse>

    <UserMode
      @pressed="pressed"
      @released="released"
      v-bind:project="project"
      ref="userMode"
      v-if="project.userMode"
    ></UserMode>
    <template v-else>
      <!-- Board toolbar -->
      <ProjectSetupBar
        ref="projectSetupBar"
        v-bind:project="project"
        @rename="renameProject"
        @change="updated"
      ></ProjectSetupBar>

      <!-- Group (switch) -->
      <b-card-group deck>
        <GroupCard
          v-for="group in project.groups.slice(0, project.switchCount)"
          :key="group.id"
          v-bind:group="group"
          @change="updated"
          @pressed="pressed"
          @released="released"
          @remove="remove"
          :border-variant="group.highlighted ? 'dark' : ''"
          style="min-width: 40rem;"
          class="mb-5"
        ></GroupCard>
      </b-card-group>
    </template>

    <Presenter
      v-bind:project="project"
      @pressed="pressed"
      @released="released"
      ref="presenter"
    ></Presenter>
  </b-container>
</template>

<script>
import Vue from "vue";
import { mapState } from "vuex";

import { BootstrapVue } from "bootstrap-vue";

import LongPressCollapse from "./LongPressCollapse";
import FileManagerToolbar from "./FileManagerToolbar";
import UserMode from "./UserMode";
import ProjectSetupBar from "./ProjectSetupBar";
import GroupCard from "./GroupCard";
import Presenter from "./Presenter";

Vue.use(BootstrapVue);

export default {
  components: {
    LongPressCollapse,
    FileManagerToolbar,
    UserMode,
    ProjectSetupBar,
    GroupCard,
    Presenter
  },
  props: ["projectName"],
  computed: mapState(["project"]),
  watch: {
    "project.name": function(projectName) {
      const routeProjectName = this.$route.params.projectName || "";
      console.log(
        "Watch project name " + routeProjectName + " -> " + projectName
      );
      if (projectName && projectName.length > 0) {
        document.title = projectName + " - My Way";
      } else {
        document.title = "My way";
      }
      if (projectName != routeProjectName) {
        this.$router.replace({ path: `/project/${projectName}` });
      }
      this.$refs.projectSetupBar.updateProject(projectName);
    }
  },
  mounted() {
    console.log("Mounted: " + this.$route.params.projectName);
    this.$refs.fileManagerToolbar.loadProject(
      this.$route.params.projectName || "",
      /* isInitial= */ true
    );
  },
  methods: {
    updated() {
      this.$refs.fileManagerToolbar.saveProject(this.project);
      this.$refs.presenter.update();
    },
    remove(groupIndex) {
      this.$refs.fileManagerToolbar.spliceGroup(groupIndex);
      this.updated();
    },
    pressed(groupIndex) {
      this.$refs.presenter.handlePress(groupIndex);
      if (this.$refs.userMode) {
        this.$refs.userMode.handlePress(groupIndex);
      }
    },
    released(groupIndex) {
      this.$refs.presenter.handleRelease(groupIndex);
    },
    renameProject(project, projectName) {
      this.$refs.fileManagerToolbar.renameProject(project, projectName);
    },
    toggleUserMode() {
      this.updated();
    }
  }
};
</script>
