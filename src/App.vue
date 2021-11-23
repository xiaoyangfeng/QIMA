<template>
  <el-container direction="vertical">
    <div>
      <span
        >Current status is
        <strong class="status-name"
          >{{ currentStatus || "empty" }}
        </strong></span
      >
      <el-button @click="showCreateStatus">Create Status</el-button>
    </div>
    <div>
      <span>Update status: </span>
      <el-select
        placeholder="Select a status"
        v-model="selectedStatusForUpdate"
        @change="showUpdateStatus"
      >
        <el-option
          v-for="status in statuses"
          :key="status.name"
          :value="status.name"
        >
        </el-option> </el-select
      >?
    </div>
    <div>
      <span>Could be transferred to </span>
      <el-select
        placeholder="Select a status"
        v-model="selectedStatusForAbility"
        @change="canTransferStatus"
      >
        <el-option
          v-for="status in statuses"
          :key="status.name"
          :value="status.name"
        >
        </el-option> </el-select
      >?
      <div v-show="messageForAbility">{{ messageForAbility }}</div>
    </div>
    <div>
      <span>How could be transferred to </span>
      <el-select
        placeholder="Select a status"
        v-model="selectedStatusForHow"
        @change="howToTransferStatus"
      >
        <el-option
          v-for="status in statuses"
          :key="status.name"
          :value="status.name"
        >
        </el-option> </el-select
      >?
      <div v-show="messageForHow">{{ messageForHow }}</div>
    </div>
    <div>
      <span>Transferred to </span>
      <el-select
        placeholder="Select a status"
        v-model="selectedStatusForTransfer"
        @change="beforeTransferStatus"
      >
        <el-option
          v-for="status in statuses"
          :key="status.name"
          :value="status.name"
        >
        </el-option> </el-select
      >?
      <div v-show="messageForTransfer">{{ messageForTransfer }}</div>
    </div>
    <el-dialog :visible.sync="createStatusDialogVisible" width="600px">
      <el-link type="primary" @click="createStatusDialogVisible = false"
        >main page</el-link
      >
      <el-form v-model="statusToAdd" label-width="200px">
        <el-form-item label="Status Name: " prop="name">
          <el-input v-model="statusToAdd.name"></el-input>
        </el-form-item>
        <el-form-item label="Could be transferred to: " prop="validTransferTo">
          <el-checkbox-group v-model="statusToAdd.validTransferTo">
            <el-checkbox label="self"></el-checkbox>
            <el-checkbox
              v-for="status in statuses"
              :key="status.name"
              :label="status.name"
            ></el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item>
          <el-button @click="createStatus">Create Status</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <el-dialog
      :visible.sync="updateStatusDialogVisible"
      width="600px"
      @close="handleBeforeClose"
    >
      <el-link type="primary" @click="updateStatusDialogVisible = false"
        >main page</el-link
      >
      <el-form v-model="statusToModify" label-width="200px">
        <el-form-item label="Status Name: ">
          <span>{{ selectedStatusForUpdate }} </span>
          <el-button
            v-if="selectedStatusForUpdate !== currentStatus"
            @click="deleteStatus"
            >delete</el-button
          >
        </el-form-item>
        <el-form-item label="Could be transferred to: " prop="validTransferTo">
          <el-checkbox-group v-model="statusToModify.validTransferTo">
            <el-checkbox
              v-for="status in statuses"
              :key="status.name"
              :label="
                selectedStatusForUpdate === status.name ? 'self' : status.name
              "
            ></el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item>
          <el-button @click="updateStatus">Update Status</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </el-container>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      messageForAbility: "",
      messageForHow: "",
      messageForTransfer: "",
      paths: {},
      visited: {},
      currentStatus: "",
      selectedStatusForUpdate: "",
      selectedStatusForAbility: "",
      selectedStatusForHow: "",
      selectedStatusForTransfer: "",
      statuses: {},
      createStatusDialogVisible: false,
      updateStatusDialogVisible: false,
      statusToAdd: {
        name: "",
        validTransferTo: [],
        validTransferFrom: [],
      },
      statusToModify: {
        name: "",
        validTransferTo: [],
        validTransferFrom: [],
      },
    };
  },
  methods: {
    cleanPath() {
      this.visited = {};
      this.paths = {};
    },
    handleBeforeClose() {
      this.selectedStatusForUpdate = "";
    },
    canTransferStatus() {
      if (!this.currentStatus) {
        this.messageForAbility = `Yes, can be transferred to ${this.selectedStatusForAbility} from empty`;
        return;
      }
      this.cleanPath();
      let result = this.minPath(
        this.currentStatus,
        this.selectedStatusForAbility
      );
      if (result) {
        this.messageForAbility = `Yes, can be transferred to ${this.selectedStatusForAbility} from ${this.currentStatus}`;
      } else {
        this.messageForAbility = `Can not be transferred to ${this.selectedStatusForAbility}`;
      }
    },
    howToTransferStatus() {
      if (!this.currentStatus) {
        this.messageForHow = `=> ${this.selectedStatusForHow}`;
        return;
      }
      this.cleanPath();
      let result = this.minPath(this.currentStatus, this.selectedStatusForHow);
      if (result) {
        this.messageForHow = result.join(" => ");
      } else {
        this.messageForHow = `Can not be transferred to ${this.selectedStatusForHow}`;
      }
    },
    beforeTransferStatus() {
      if (!this.currentStatus) {
        this.transferStatus();
        return;
      }
      this.cleanPath();
      let result = this.statuses[this.currentStatus].validTransferTo.some(
        (item) => {
          return item === "self"
            ? this.selectedStatusForTransfer === this.currentStatus
            : item === this.selectedStatusForTransfer;
        }
      );
      if (result) {
        this.transferStatus();
      } else {
        this.messageForTransfer = `Can not be transferred to ${this.selectedStatusForTransfer}`;
      }
    },
    transferStatus() {
      this.currentStatus = this.selectedStatusForTransfer;
      this.selectedStatusForAbility = "";
      this.messageForAbility = "";
      this.selectedStatusForHow = "";
      this.messageForHow = "";
      this.selectedStatusForTransfer = "";
      this.messageForTransfer = "";
    },
    deleteStatus() {
      this.updateStatusDialogVisible = false;
      this.statuses[this.selectedStatusForUpdate].validTransferFrom.forEach(
        (item) => {
          if (item === "self") return;
          let index = this.statuses[item].validTransferTo.indexOf(
            this.statusToModify.name
          );
          if (index !== -1) {
            this.statuses[item].validTransferTo.splice(index, 1);
          }
        }
      );
      this.statuses[this.selectedStatusForUpdate].validTransferTo.forEach(
        (item) => {
          if (item === "self") return;
          let index = this.statuses[item].validTransferFrom.indexOf(
            this.statusToModify.name
          );
          if (index !== -1) {
            this.statuses[item].validTransferFrom.splice(index, 1);
          }
        }
      );
      delete this.statuses[this.selectedStatusForUpdate];
    },
    showCreateStatus() {
      this.createStatusDialogVisible = true;
    },
    showUpdateStatus(name) {
      this.statusToModify = { ...this.statuses[name] };
      this.updateStatusDialogVisible = true;
    },
    createStatus() {
      if (this.statuses[this.statusToAdd.name]) {
        this.$message.error("This status name has already existed");
        return;
      }
      this.createStatusDialogVisible = false;
      this.statuses[this.statusToAdd.name] = { ...this.statusToAdd };
      this.statusToAdd.validTransferTo.forEach((item) => {
        if (item === "self") return;
        if (
          this.statuses[item].validTransferFrom.indexOf(
            this.statusToAdd.name
          ) === -1
        ) {
          this.statuses[item].validTransferFrom.push(this.statusToAdd.name);
        }
      });
      Object.assign(this.statusToAdd, this.$options.data().statusToAdd);
    },
    updateStatus() {
      this.updateStatusDialogVisible = false;
      this.statuses[this.statusToModify.name] = { ...this.statusToModify };
      this.statusToModify.validTransferTo.forEach((item) => {
        if (item === "self") return;
        if (
          this.statuses[item].validTransferFrom.indexOf(
            this.statusToModify.name
          ) === -1
        ) {
          this.statuses[item].validTransferFrom.push(this.statusToModify.name);
        }
      });
      Object.assign(this.statusToModify, this.$options.data().statusToModify);
    },
    minPath(start, end) {
      this.visited[start] = true;
      if (this.paths[start]) {
        return this.paths[start];
      }
      this.statuses[start].validTransferTo.forEach((item) => {
        item = item === "self" ? start : item;
        if (item === end) {
          this.paths[start] = [start, end];
        } else if (!this.visited[item]) {
          let result = this.minPath(item, end);
          if (result) {
            if (
              !this.paths[start] ||
              result.length < this.paths[start].length
            ) {
              this.paths[start] = [start, ...result];
            }
          }
        }
      });
      return this.paths[start];
    },
  },
};
</script>

<style scoped>
.el-container {
  width: 70%;
  margin: auto;
}
.el-container > div {
  margin: 15px 0;
}
.status-name {
  font-size: 20px;
  color: orange;
}
.el-link{
  margin-bottom: 50px;
}
</style>
