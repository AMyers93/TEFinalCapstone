<template>
  <div>
    <div>
      <h2 class="label salmon">Available Prizes</h2>
      <div class="has-text-danger" v-if="formErrors">There were problems creating this prize.</div>
      <div v-if="noPrizes">There are no prizes entered! Add a prize!!!</div>
      <button class="button is-info is-small" v-if="!showForm && isParent" v-on:click="showFormButton()">
        Create New Prize
      </button>
      <button class="button is-info is-small" v-if="showForm" v-on:click="hideFormButton()">Hide Form</button>
    </div>
    <br>
    <div v-if="showForm">
      <div class="columns">
        <div class="column">
          <div class="field">
            <label class="label" for="prize_name">Prize Name </label>
            <div class="control">
              <input
                class="input is-small"
                type="text"
                id="prize_name"
                name="prize_name"
                placeholder="Enter Prize Name"
                v-model="prizeinfo.prizeName"
                required
                autofocus
              />
            </div>
          </div>
        </div>

        <div class="column">
          <div class="field">
            <label class="label" for="description">Description </label>
            <div class="control">
              <input
                class="input is-small"
                type="text"
                id="description"
                name="description"
                placeholder="Enter Prize Description"
                v-model="prizeinfo.prizeDescription"
                required
                autofocus
              />
            </div>
          </div>
        </div>

        <div class="column">
          <div class="field">
            <label class="label" for="milestone"
              >Milestone (minutes reading)
            </label>
            <div class="control">
              <input
                class="input is-small"
                type="text"
                id="milestone"
                name="milestone"
                placeholder="Enter Milestone"
                v-model="prizeinfo.milestone"
                required
                autofocus
              />
            </div>
          </div>
        </div>
      </div>

      <div class="columns">
        <div class="column">
          <div class="field">
            <label for="start_date">Start Date</label>
            <div class="control">
              <input
                class="input is-small"
                type="text"
                id="start_date"
                name="start_date"
                placeholder="yyyy-mm-dd"
                v-model="prizeinfo.startDate"
                required
                autofocus
              />
            </div>
          </div>
        </div>
        <div class="column">
          <div class="field">
            <label for="end_date">End Date</label>
            <div class="control">
              <input
                class="input is-small"
                type="text"
                id="end_date"
                name="end_date"
                placeholder="yyyy-mm-dd"
                v-model="prizeinfo.endDate"
                required
                autofocus
              />
            </div>
          </div>
        </div>
        <div class="column">
          <div class="field">
            <label for="max_prizes">Max Prizes Available</label>
            <div class="control">
              <input
                class="input is-small"
                type="text"
                id="numOfPrizes"
                name="numOfPrizes"
                placeholder="Enter Prize Cap"
                v-model="prizeinfo.numOfPrizes"
                required
                autofocus
              />
            </div>
          </div>
        </div>
        <div class="column">
          <div class="field">
            <label for="user_group">Parent/Child Prize</label>
            <br>
            <select id="users" v-model="prizeinfo.userGroup">
              <option :value="parent">Parent</option>
              <option :value="child">Child</option>
            </select>
          </div>
        </div>

        
      </div>
      <button
        v-on:click="savePrize()"
        v-if="this.prizeIdNum == 0"
        class="button is-info is-small"
        type="submit"
      >
        Add Prize
      </button>
      <button
        v-on:click="savePrize()"
        v-if="this.prizeIdNum != 0"
        class="button is-info is-small"
        type="submit"
      >
        Edit Prize
      </button>
    </div>

    <new-prize
      v-if="!showForm"
      v-on:editPrize="editPrize($event); showFormButton();"
      v-on:deletePrize="
        deletePrize($event);
        hideFormButton();
      "
    ></new-prize>
  </div>
</template>

<script>
import axios from "axios";
import NewPrize from "@/components/NewPrize";
import { eventBus } from "../main.js";

export default {
  components: {
    NewPrize
  },
  props: {
    apiURL: String
  },

  data() {
    return {
      parent: "user",
      child: "child",
      isParent: false,
      showForm: false,
      prizeIdNum: 0,
      prizeinfo: {
        prizeId: "",
        prizeName: "",
        prizeDescription: "",
        milestone: "",
        userGroup: "",
        numOfPrizes: "",
        startDate: "",
        endDate: ""
      },
      currentUser: {
        userId: "",
        userName: "",
        password: "",
        confirmPassword: "",
        passwordMatching: "",
        role: ""
      },
      formErrors: false,
      noPrizes: false
    };
  },

  methods: {
    savePrize() {
      this.prizeIdNum === 0 ? this.addNewPrize() : this.updatePrize();
    },
    editPrize(id) {
      this.prizeIdNum = id;
    },
    addNewPrize() {
      axios
        .post(
          `${process.env.VUE_APP_REMOTE_API}/api/addPrize`,
          this.prizeinfo,
          {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("Authorization")
            }
          }
        )
        // eslint-disable-next-line no-unused-vars
        .then(response => {
          this.showForm = false;
          eventBus.$emit("refreshCreatePrize");
          console.log("Create Prize fired");
        })
        .catch(error => {
          console.log(error + " there was an error");
        });
    },
    updatePrize() {
      axios
        .post(
          `${process.env.VUE_APP_REMOTE_API}/api/editPrize`,
          this.prizeinfo,
          {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("Authorization")
            }
          }
        )
        // eslint-disable-next-line no-unused-vars
        .then(response => {
          this.showForm = false;
          eventBus.$emit("refreshCreatePrize");
        })
        .catch(error => {
          console.log(error + " there was an error");
        });
    },
    deletePrize(id) {
      this.prizeinfo.prizeId = id;
      eventBus.$emit("refreshCreatePrize");
      axios
        .post(
          `${process.env.VUE_APP_REMOTE_API}/api/deletePrize`,
          this.prizeinfo,
          {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("Authorization")
            }
          }
        )
        // eslint-disable-next-line no-unused-vars
        .then(response => {})
        .catch(error => {
          console.log(error + " there was an error");
        });
    },

    showFormButton() {
      this.prizeinfo.prizeId = "";
      this.prizeinfo.prizeName = "";
      this.prizeinfo.prizeDescription = "";
      this.prizeinfo.milestone = "";
      this.prizeinfo.userGroup = "";
      this.prizeinfo.numOfPrizes = "";
      this.prizeinfo.startDate = "";
      this.prizeinfo.endDate = "";
      this.showForm = true;
      if (this.prizeIdNum != 0) {
        axios
          .get(
            `${process.env.VUE_APP_REMOTE_API}/api/getPrize/${this.prizeIdNum}`,
            {
              headers: {
                Authorization: "Bearer " + localStorage.getItem("Authorization")
              }
            }
          )
          .then(response => {
            this.prizeinfo = response.data;
            if (this.prizeinfo.userGroup == "user") {
              this.prizeinfo.userGroup = "Parent";
              this.isParent = true;
            }
          })
          .catch(err => console.error(err));
      }
    },
    hideFormButton() {
      this.prizeIdNum = 0;
      this.showForm = false;
    }
  },
  created() {
    axios
      .get(`${process.env.VUE_APP_REMOTE_API}/api/getCurrentUser`, {
        headers: {
          Authorization: "Bearer " + localStorage.getItem("Authorization")
        }
      })
      .then(response => {
        this.currentUser = response.data;
        if(this.currentUser.role == "user"){
          this.isParent = true;
        }
      })
      .catch(error => {
        console.log(error + " there was an error");
      });
  }
};
</script>



<style scoped>
h2 {
  background-color:gold;
}

</style>